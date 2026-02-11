# Python Backend Development Rules

**Specific to Python backend projects (FastAPI, Flask, Django)**

---

## 1. Core Architecture & File Structure

### Modular Separation
```
project/
├── api_service/          # FastAPI routes, Pydantic models
│   ├── routes/           # API endpoints
│   ├── models/           # Request/response schemas
│   └── services/         # Business logic
├── llm_source/           # LLM implementations
├── utils/                # Shared utilities
├── tests/                # Test files
├── constant_var.py       # Central config hub
├── env.py                # Environment variables
├── config.py             # Application config
└── requirements.txt
```

### Rules:
* **One Class = One File:** Each class in its own file
* **Centralized Config:** NEVER call `os.getenv` directly in business logic
    - Always import from `constant_var.py`
* **Clear Separation:** Routes handle HTTP, Services handle logic, Utils handle helpers

---

## 2. Strict Typing & Datatypes (Non-Negotiable)

### Universal Type Hints
```python
# ❌ Bad: No types
def process_data(data):
    count = 0
    return data

# ✅ Good: Complete types
from typing import Dict, Any

def process_data(data: Dict[str, Any]) -> Dict[str, Any]:
    """
    Process incoming data payload.
    
    Args:
        data: Raw data dictionary
        
    Returns:
        Processed data dictionary
    """
    count: int = 0
    result: Dict[str, Any] = {}
    return result
```

### Explicit Variable Declaration
```python
# ❌ Incorrect
count = 0
users = []
config = None

# ✅ Correct
count: int = 0
users: list[str] = []
config: dict[str, Any] | None = None
```

### Pydantic for Data Models
```python
# ✅ Preferred over raw dicts
from pydantic import BaseModel, Field

class User(BaseModel):
    id: str
    name: str = Field(..., min_length=1, max_length=100)
    email: str
    age: int = Field(..., ge=18)
```

---

## 3. Documentation Standards

### Google-Style Docstrings (Mandatory)
```python
def fetch_user_data(user_id: str, include_posts: bool = False) -> dict[str, Any]:
    """
    Fetch user data from database with optional posts.
    
    Args:
        user_id: Unique identifier for the user
        include_posts: Whether to include user's posts in response
        
    Returns:
        Dictionary containing user data and optionally posts
        
    Raises:
        UserNotFoundError: If user ID doesn't exist
        DatabaseError: If database connection fails
    """
    # Implementation...
```

### Requirements:
* Every class must have docstring
* Every public method must have docstring
* Include Args, Returns, Raises sections

---

## 4. Logging Strategy (Critical)

### NO `print()` Statements
```python
# ❌ Forbidden
print("Processing user data")
print(f"Error: {error}")

# ✅ Use project loggers
from constant_var import debug_info, detail_debug, debug_error

debug_info("[Service] Processing user data")
detail_debug(f"[Service] User ID: {user_id}")
debug_error(f"[Service] Error occurred: {str(error)}")
```

### Logging Functions from `constant_var.py`
* `debug_info("msg")` → General info (production logs)
* `detail_debug("msg")` → Verbose debugging (details.log only)
* `debug_prompt("msg")` → Raw LLM inputs
* `debug_llm_response(...)` → Raw LLM outputs
* `debug_error("msg")` → Error logging
* `debug_warning("msg")` → Warning logging

### Timezone
* All logs use **Asia/Jakarta** timezone

---

## 5. Error Handling & Resilience

### Defensive Parsing
```python
# ❌ Bad: Direct JSON parsing
import json
result = json.loads(llm_output)

# ✅ Good: Safe parsing
from utils.safe_parsers import SafeJSONParser

parser: SafeJSONParser = SafeJSONParser()
result: dict[str, Any] = parser.parse(llm_output)

if not result:
    detail_debug("Failed to parse LLM response")
    return {}
```

### Result Pattern
```python
from typing import NamedTuple

class ParseResult(NamedTuple):
    success: bool
    data: dict[str, Any]
    error: str | None

def parse_llm_output(output: str) -> ParseResult:
    """Parse LLM output safely."""
    try:
        data = json.loads(output)
        return ParseResult(success=True, data=data, error=None)
    except json.JSONDecodeError as e:
        return ParseResult(success=False, data={}, error=str(e))
```

### Fail Fast with Guard Clauses
```python
def process_user(user_id: str, data: dict) -> bool:
    """Process user data."""
    # Guard clauses at the top
    if not user_id:
        debug_error("[Service] Missing user_id")
        return False
    
    if not data:
        debug_error("[Service] No data provided")
        return False
    
    # Main logic here
    # ...
```

---

## 6. Coding Conventions (PEP 8+)

### Naming Conventions
* **Classes:** `PascalCase` (e.g., `UserService`, `AuthMiddleware`)
* **Functions/Variables:** `snake_case` (e.g., `get_user_data`, `user_id`)
* **Constants:** `UPPER_CASE` (e.g., `API_KEY`, `MAX_RETRIES`)
* **Private methods:** `_snake_case` (e.g., `_validate_input`)

### Clean Code Rules
```python
# ❌ Bad: Dead code, commented blocks
import os  # unused
from typing import Any

def process():
    # old_logic()
    # return None
    pass

# ✅ Good: Clean, focused
from typing import Any
from constant_var import debug_info

def process_data(data: dict[str, Any]) -> bool:
    """Process incoming data."""
    debug_info("[Service] Processing data")
    return True
```

### DRY Principle
```python
# ❌ Bad: Repeated logic
def validate_email(email: str) -> bool:
    return '@' in email and '.' in email

def validate_user_email(user: User) -> bool:
    return '@' in user.email and '.' in user.email

# ✅ Good: Extracted to helper
def is_valid_email(email: str) -> bool:
    """Check if email format is valid."""
    return '@' in email and '.' in email

def validate_user(user: User) -> bool:
    """Validate user data."""
    return is_valid_email(user.email)
```

---

## 7. AI & LangChain Policy

### Pragmatism First
* Use **LangChain** only if it genuinely simplifies the solution
    - Good use cases: Complex chains, memory management, prompt templates
    - Avoid for: Simple API calls, basic text processing

### Plain Python Fallback
```python
# ✅ Preferred for simple cases
import httpx

async def call_llm(prompt: str) -> str:
    """Call LLM API directly."""
    async with httpx.AsyncClient() as client:
        response = await client.post(
            "https://api.provider.com/v1/chat",
            json={"prompt": prompt}
        )
        return response.json()["response"]

# ⚠️ Only use LangChain when needed
from langchain.chains import ConversationChain
from langchain.memory import ConversationBufferMemory

def create_conversation_chain():
    """Use LangChain for complex conversation management."""
    memory = ConversationBufferMemory()
    chain = ConversationChain(llm=llm, memory=memory)
    return chain
```

---

## 8. Async/Await Best Practices

### All I/O Operations Must Be Async
```python
# ❌ Bad: Blocking I/O
import requests

def get_user(user_id: str) -> dict:
    response = requests.get(f"/api/users/{user_id}")
    return response.json()

# ✅ Good: Async I/O
import httpx

async def get_user(user_id: str) -> dict[str, Any]:
    """Fetch user data asynchronously."""
    async with httpx.AsyncClient(timeout=10.0) as client:
        response: httpx.Response = await client.get(f"/api/users/{user_id}")
        response.raise_for_status()
        return response.json()
```

### Database Queries
```python
# ✅ Good: Async database operations
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select

async def get_user_from_db(db: AsyncSession, user_id: str) -> User | None:
    """Fetch user from database."""
    result = await db.execute(
        select(User).where(User.id == user_id)
    )
    return result.scalar_one_or_none()
```

---

## 9. FastAPI Best Practices

### Complete Endpoint Example
```python
from typing import Optional
from fastapi import APIRouter, Header, HTTPException, status, Depends
from pydantic import BaseModel, Field

from config import APIConfig
from constant_var import debug_info, debug_error
from services.user_service import UserService

router = APIRouter(tags=["Users"])


class CreateUserRequest(BaseModel):
    """Request model for creating user."""
    email: str = Field(..., min_length=5, max_length=100)
    name: str = Field(..., min_length=1, max_length=100)
    age: int = Field(..., ge=18, le=120)


class UserResponse(BaseModel):
    """Response model for user data."""
    id: str
    email: str
    name: str
    age: int


def verify_api_key(x_api_key: Optional[str] = Header(None)) -> str:
    """Verify API key from header."""
    if x_api_key is None:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Missing API key"
        )
    
    if x_api_key != APIConfig.API_KEY:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Invalid API key"
        )
    
    return x_api_key


@router.post("/users", response_model=UserResponse, status_code=201)
async def create_user(
    request: CreateUserRequest,
    api_key: str = Depends(verify_api_key)
) -> UserResponse:
    """
    Create a new user.
    
    Args:
        request: User data for creation
        api_key: Validated API key from header
        
    Returns:
        Created user data
    """
    debug_info(f"[UserAPI] Creating user: {request.email}")
    
    service: UserService = UserService()
    
    try:
        user: User = await service.create_user(
            email=request.email,
            name=request.name,
            age=request.age
        )
        
        return UserResponse(
            id=user.id,
            email=user.email,
            name=user.name,
            age=user.age
        )
        
    except Exception as e:
        debug_error(f"[UserAPI] Error creating user: {str(e)}")
        raise HTTPException(
            status_code=500,
            detail="Failed to create user"
        )
```

---

## 10. Testing Standards

### pytest + httpx Pattern
```python
import pytest
from httpx import AsyncClient
from config import APIConfig

@pytest.mark.asyncio
async def test_create_user():
    """Test user creation endpoint."""
    async with AsyncClient(app=app, base_url="http://test") as client:
        # Test successful creation
        response = await client.post(
            "/api/users",
            headers={
                "Content-Type": "application/json",
                "X-API-Key": APIConfig.API_KEY
            },
            json={
                "email": "test@example.com",
                "name": "Test User",
                "age": 25
            }
        )
        
        assert response.status_code == 201
        data = response.json()
        assert data["email"] == "test@example.com"
        assert "id" in data

@pytest.mark.asyncio
async def test_create_user_invalid_age():
    """Test validation for underage user."""
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.post(
            "/api/users",
            headers={"X-API-Key": APIConfig.API_KEY},
            json={"email": "test@example.com", "name": "Test", "age": 17}
        )
        
        assert response.status_code == 422  # Validation error
```

### Coverage Requirements
* Minimum 80% code coverage
* 100% coverage for critical paths (auth, payments, data mutations)

---

## 11. Dependency Management

### requirements.txt Best Practices
```txt
# Core framework
fastapi==0.104.1
uvicorn[standard]==0.24.0
pydantic==2.5.0

# Database
sqlalchemy==2.0.23
asyncpg==0.29.0

# HTTP client
httpx==0.25.2

# Utilities
python-dotenv==1.0.0
pydantic-settings==2.1.0

# Development
pytest==7.4.3
pytest-asyncio==0.21.1
black==23.11.0
ruff==0.1.6
```

### Pin Version Numbers
* Always specify exact versions in production
* Use `pip freeze > requirements.txt` after testing

---

## 12. Environment Variables Pattern

### env.py Structure
```python
import os
from dotenv import load_dotenv

load_dotenv()

class ConstantsVar:
    """Central environment variables."""
    
    # API Keys
    OPENAI_API_KEY: str = os.getenv("OPENAI_API_KEY", "")
    GEMINI_API_KEY: str = os.getenv("GEMINI_API_KEY", "")
    
    # Database
    DATABASE_URL: str = os.getenv("DATABASE_URL", "")
    
    # Supabase
    SUPABASE_PROJECT_URL: str = os.getenv("SUPABASE_PROJECT_URL", "")
    SUPABASE_ANON_KEY: str = os.getenv("SUPABASE_ANON_KEY", "")
    
    # Application
    NFL_CHATBOT_API_KEY: str = os.getenv("NFL_CHATBOT_API_KEY", "")
```

### config.py Structure
```python
from typing import ClassVar
from env import ConstantsVar

class APIConfig:
    """Application configuration."""
    
    # API Settings
    API_KEY: ClassVar[str] = ConstantsVar.NFL_CHATBOT_API_KEY
    
    # Supabase
    SUPABASE_URL: ClassVar[str] = ConstantsVar.SUPABASE_PROJECT_URL
    SUPABASE_KEY: ClassVar[str] = ConstantsVar.SUPABASE_ANON_KEY
    
    # LLM
    OPENAI_KEY: ClassVar[str] = ConstantsVar.OPENAI_API_KEY
```

---

## Summary: Python Backend Best Practices

1. **Type Everything** — Explicit types = fewer bugs
2. **Log, Don't Print** — Use `debug_info`, `detail_debug`, etc.
3. **Centralize Config** — Import from `constant_var.py`, not `os.getenv`
4. **Async All I/O** — Database, HTTP, file operations
5. **Validate Inputs** — Pydantic models with Field validation
6. **Handle Errors** — Safe parsing, Result pattern, guard clauses
7. **Document Code** — Google-style docstrings everywhere
8. **Test Thoroughly** — pytest + httpx, 80%+ coverage
9. **Keep Clean** — Remove dead code, follow PEP 8, DRY
10. **Pin Dependencies** — Exact versions in requirements.txt

**Remember:** Production-ready code is typed, tested, logged, and documented.

# Python Backend Development Rules

**Extends:** [Universal Principles](./universal-principles.md)  
**Focus:** FastAPI, async services, type safety, serverless deployment

---

## 1. Project Architecture

### Standard Structure
```
project/
├─ api_service/     # FastAPI app, Pydantic models, business logic
├─ llm_source/      # LLM implementations (Gemini, OpenAI, etc.)
├─ routes/          # API route handlers (HTTP layer only)
├─ services/        # Business logic services
├─ models/          # Pydantic schemas, data models
├─ utils/           # Shared utilities (parsing, validation, etc.)
├─ config.py        # Configuration management
├─ env.py           # Environment variable loading
└─ app.py           # FastAPI application entry point
```

### File Organization
* **One Class = One File:** Each class in its own file
* **Centralized Config:** Never `os.getenv()` in business logic - always use `config.py`
* **Service Layer Pattern:**
    - `routes/` → HTTP layer (validation, headers, status codes)
    - `services/` → Business logic (LLM, database, external APIs)
    - `utils/` → Pure functions (parsing, formatting, helpers)

---

## 2. Strict Typing (Non-Negotiable)

### Type Hints Everywhere
```python
# ❌ BAD
def process_data(data):
    count = 0
    users = []
    return count

# ✅ GOOD
from typing import Any

def process_data(data: dict[str, Any]) -> int:
    count: int = 0
    users: list[str] = []
    return count
```

### Rules
* All function parameters → type hints
* All return values → type hints
* All class attributes → type hints
* All local variables → explicit type annotation at initialization
* Use Pydantic `BaseModel` for structured data (not raw dicts)

---

## 3. Documentation Standards

### Google-Style Docstrings (Mandatory)
```python
def fetch_user_data(user_id: str, include_metadata: bool = False) -> Optional[dict[str, Any]]:
    """
    Fetch user data from database.
    
    Args:
        user_id: Unique user identifier
        include_metadata: Whether to include additional metadata
        
    Returns:
        User data dictionary if found, None otherwise
        
    Raises:
        DatabaseError: If database connection fails
    """
    pass
```

### Required Sections
* Description: Concise summary
* Args: Parameter name + type + meaning
* Returns: Type + meaning
* Raises: Exceptions and side effects

---

## 4. Logging Strategy

### Use Project Loggers (NO `print()`)
```python
from config import debug_info, debug_error, detail_debug

# ❌ BAD
print(f"Processing user {user_id}")

# ✅ GOOD
debug_info(f"[Service] Processing user {user_id}")
detail_debug(f"[Service] User data: {user_data}")
debug_error(f"[Service] Failed to fetch user: {str(e)}")
```

### Logger Types
* `debug_info()` → General info (production logs)
* `detail_debug()` → Verbose debugging (`details.log` only)
* `debug_prompt()` → Raw LLM inputs
* `debug_llm_response()` → Raw LLM outputs
* `debug_error()` → Error messages
* **Timezone:** All logs in Asia/Jakarta

---

## 5. Error Handling

### Defensive Parsing
```python
# ❌ BAD
import json
result = json.loads(llm_response)

# ✅ GOOD
from utils.safe_parsers import SafeJSONParser

parser: SafeJSONParser = SafeJSONParser()
result: dict[str, Any] = parser.parse(llm_response)

if not result:
    detail_debug("[Service] Failed to parse LLM response")
    return {}
```

### Result Pattern (Prefer over Exceptions)
```python
from typing import NamedTuple

class ParseResult(NamedTuple):
    success: bool
    data: dict[str, Any]
    error: str | None

def parse_response(text: str) -> ParseResult:
    try:
        data = safe_parse(text)
        return ParseResult(success=True, data=data, error=None)
    except Exception as e:
        return ParseResult(success=False, data={}, error=str(e))
```

---

## 6. FastAPI Best Practices

### Endpoint Structure
```python
from typing import Optional
from fastapi import APIRouter, Header, HTTPException, status
from pydantic import BaseModel, Field

from config import APIConfig, debug_info, debug_error

router = APIRouter(tags=["Users"])


class UserRequest(BaseModel):
    """Request validation."""
    user_id: str = Field(..., min_length=1, max_length=100)
    email: str = Field(..., regex=r"^[\w\.\-]+@[\w\.\-]+\.\w+$")


class UserResponse(BaseModel):
    """Response schema."""
    status: str
    data: Optional[dict] = None
    error: Optional[str] = None


async def verify_api_key(x_api_key: Optional[str] = Header(None)) -> str:
    """Verify API key from header."""
    if x_api_key is None or x_api_key != APIConfig.API_KEY:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid or missing API key"
        )
    return x_api_key


@router.post("/users", response_model=UserResponse)
async def create_user(
    request: UserRequest,
    x_api_key: str = Header(..., description="API Key")
) -> UserResponse:
    """
    Create new user endpoint.
    
    Args:
        request: Validated user data
        x_api_key: API authentication key
        
    Returns:
        UserResponse with creation result
    """
    verify_api_key(x_api_key)
    
    debug_info(f"[Users] Creating user: {request.user_id}")
    
    try:
        # Business logic here
        return UserResponse(status="success", data={"user_id": request.user_id})
    except Exception as e:
        debug_error(f"[Users] Error: {str(e)}")
        return UserResponse(status="error", error=str(e))
```

---

## 7. Async/Await (Serverless Critical)

### All I/O Must Be Async
```python
import httpx
from typing import Dict, Any, Optional

# ❌ BAD (Blocking I/O)
def get_data(url: str):
    response = requests.get(url)
    return response.json()

# ✅ GOOD (Async I/O)
async def get_data(url: str) -> Optional[Dict[str, Any]]:
    """Fetch data from external API."""
    try:
        async with httpx.AsyncClient(timeout=10.0) as client:
            response: httpx.Response = await client.get(url)
            response.raise_for_status()
            return response.json()
    except httpx.HTTPError as e:
        debug_error(f"[Service] HTTP error: {str(e)}")
        return None
```

### Connection Pooling
* Reuse `httpx.AsyncClient` instances
* Use database connection pools (`asyncpg`)
* Lazy load heavy dependencies

---

## 8. Deployment (Vercel/Serverless)

### Size Limit Awareness (~50MB)
**❌ Avoid Heavy Packages:**
* LangChain (~100MB+)
* TensorFlow, PyTorch (500MB+)
* pandas with dependencies (~100MB)

**✅ Prefer Lightweight:**
* `httpx` over `requests`
* Direct SDKs over framework wrappers
* Built-in libraries when possible

**Check Before Adding:**
```bash
pip show <package> | grep "Size:"
```

### Environment Variables
```python
# ❌ BAD
api_key = "hardcoded-secret-key"

# ✅ GOOD (in env.py)
import os
from dotenv import load_dotenv

load_dotenv()

class ConstantsVar:
    API_KEY: str = os.getenv("NFL_CHATBOT_API_KEY", "")
    DATABASE_URL: str = os.getenv("DATABASE_URL", "")
```

### Cold Start Optimization
* Minimize global imports
* Lazy load heavy dependencies
* Keep initialization light

---

## 9. Testing

### Test Structure
```python
"""
Test script for User endpoints.
"""

import httpx
import asyncio
from config import APIConfig

BASE_URL = "http://localhost:8000"


async def test_user_endpoints():
    """Comprehensive endpoint testing."""
    
    print("=== Testing User Endpoints ===\n")
    
    async with httpx.AsyncClient(timeout=30.0) as client:
        # Test 1: Happy path
        print("1. Testing user creation...")
        response = await client.post(
            f"{BASE_URL}/api/users",
            headers={"X-API-Key": APIConfig.API_KEY},
            json={"user_id": "test001", "email": "test@example.com"}
        )
        
        assert response.status_code == 200
        print(f"   ✅ Status: {response.status_code}\n")
        
        # Test 2: Invalid data
        print("2. Testing validation...")
        response2 = await client.post(
            f"{BASE_URL}/api/users",
            headers={"X-API-Key": APIConfig.API_KEY},
            json={"user_id": "", "email": "invalid"}
        )
        
        assert response2.status_code == 422  # Validation error
        print(f"   ✅ Validation working\n")
        
        # Test 3: Auth check
        print("3. Testing authentication...")
        response3 = await client.post(
            f"{BASE_URL}/api/users",
            headers={"X-API-Key": "wrong_key"},
            json={"user_id": "test002", "email": "test2@example.com"}
        )
        
        assert response3.status_code == 401
        print(f"   ✅ Auth working\n")
        
        print("=== All tests passed! ===")


if __name__ == "__main__":
    asyncio.run(test_user_endpoints())
```

---

## 10. Code Conventions (PEP 8+)

### Naming
* Classes: `PascalCase` (e.g., `UserService`)
* Functions/Variables: `snake_case`
* Constants: `UPPER_CASE`
* Private members: `_leading_underscore`

### Clean Code
* Remove unused imports immediately
* Delete commented-out code
* Extract repeated logic (DRY)
* Maximum line length: 100 characters

---

## Summary: Python Backend Checklist

✅ Strict type hints on everything  
✅ Google-style docstrings  
✅ Project loggers (no `print()`)  
✅ Async/await for all I/O  
✅ Pydantic models for data validation  
✅ Error handling with Result pattern  
✅ Lightweight dependencies (<50MB total)  
✅ FastAPI best practices (validation, auth, response models)  
✅ Comprehensive tests (3-5 cases minimum)  
✅ Environment-based configuration

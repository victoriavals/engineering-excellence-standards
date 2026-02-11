# Global Engineering Rules (Multi-Language)

**Purpose:** Comprehensive engineering standards for all development work across languages.  
**Languages Covered:** Python, JavaScript/TypeScript, HTML/CSS, Go, PHP  
**Philosophy:** Universal principles + Language-specific best practices

---

## Quick Reference

- **[Part A: Universal Principles](#part-a-universal-principles)** — Apply to ALL languages
- **[Part B: Frontend Development](#part-b-frontend-development)** — HTML/CSS/JS/TS (Priority: Deployment)
- **[Part C: Backend Development](#part-c-backend-development)** — Python/Go/PHP (Priority: API Design)

---

# Part A: Universal Principles

## 1. Verification & Testing Protocol (CRITICAL)

### Syntax/Lint Checks
**After EVERY file modification, run appropriate checks:**

**Python:**
```bash
python -c "import ast; ast.parse(open('file.py', encoding='utf-8').read()); print('file.py: OK')"
```

**JavaScript/TypeScript:**
```bash
npm run lint          # ESLint
npx tsc --noEmit     # TypeScript type check
```

**Go:**
```bash
go fmt ./...
go vet ./...
```

**PHP:**
```bash
php -l file.php      # Syntax check
./vendor/bin/phpstan analyse  # Static analysis
```

### Test Requirements
**For EVERY new feature:**
- Minimum 3-5 test cases (happy path + edge cases)
- Integration tests with dependencies (DB, APIs, services)
- Exit code 0 = success

**Testing Frameworks:**
- Python: `pytest`, `httpx.AsyncClient`
- JavaScript/TypeScript: `Jest`, `Vitest`, `Playwright`
- Go: `testing` package, `testify`
- PHP: `PHPUnit`

### Production Readiness Checklist
Before marking any task complete:
- ✅ Syntax/lint checks passed
- ✅ All tests passed (exit code 0)
- ✅ Server/build logs clean (no errors)
- ✅ Documentation updated (README, inline comments)
- ✅ Temporary test files cleaned up

---

## 2. Confirmation & Communication (Moderate Policy)

### Language Preference
- **Bahasa Indonesia** untuk komunikasi
- **English** untuk technical terms, code comments, error messages

### Auto-Proceed vs Ask for Confirmation

#### ✅ Auto-Proceed (Simple Tasks)
Execute immediately:
- Bug fixes (syntax, logic errors, typos)
- Code quality (lint fixes, docstrings/comments, unused imports)
- Documentation updates
- Test scripts (after feature approved)
- Minor refactoring (≤10 lines affected)

**Action:** Analyze → Fix → Check → Report "✅ Fixed"

#### ❓ Ask with Best Practice Recommendations (Complex Tasks)
- New features (endpoints, components, modules)
- Technology choices (multiple viable options)
- Architectural changes (structure, patterns, database)
- API contract changes
- Core system modifications

**Action:** Analyze → Research → Present 2-3 options → Recommend → Ask "Mau proceed?"

### Recommendation Format Template
```markdown
Saya ada [N] opsi untuk [task]:

**Opsi 1 (Recommended): [Name]**
✅ Pro: [Benefit 1]
✅ Pro: [Benefit 2]
❌ Con: [Drawback 1]

**Opsi 2: [Alternative]**
✅ Pro: [Benefit 1]
❌ Con: [Drawback 1]

**Best Practice Recommendation:**
Saya recommend Opsi 1 karena:
- [Reason 1: e.g., proven, lightweight, faster]
- [Reason 2]

**Pertanyaan:**
1. [Clarification 1]
2. [Clarification 2]

Mau proceed dengan Opsi 1?
```

### Implementation Plans
**For complex features (>3 files modified):**
1. Create `implementation_plan.md` with:
   - Goal & context
   - Technology rationale
   - File-by-file changes
   - Verification plan
   - Alternatives considered
2. Request review: `notify_user(PathsToReview=[plan], BlockedOnUser=true)`
3. Proceed ONLY after "LGTM"/"OK"/"Ya"

---

## 3. Workflow Patterns

### Pattern 1: Feature Implementation
1. **Planning:**
   - Ask clarifying questions (tech choices, scale, requirements)
   - Create implementation plan
   - Get approval

2. **Execution:**
   - Create task.md checklist
   - Mark `[/]` when starting, `[x]` when done
   - Run checks after each file
   - Update task_boundary every major step

3. **Verification:**
   - Create comprehensive test script
   - Run all tests
   - Check logs
   - Create walkthrough.md

4. **Completion:**
   - All checks passed
   - Tests passed
   - Logs clean
   - Docs updated
   - Temp files removed

### Pattern 2: Error Handling
**Always log errors, never silent failures:**

```python
# ❌ Bad
try:
    result = operation()
except:
    pass

# ✅ Good
try:
    result = operation()
except SpecificError as e:
    log_error(f"Error in operation: {e}")
    return fallback
```

---

## 4. Code Organization

### Universal Principles
- **Reuse Over Reinvent:** Check existing code before creating new
- **DRY:** Extract duplicated logic (>10 lines)
- **Clean After Complete:** Delete temp files, debug code, unused imports
- **Git Discipline:** Conventional commits
  - `feat: add user authentication`
  - `fix: resolve payment processing bug`
  - `docs: update API documentation`
  - `refactor: extract validation logic`

### Service Layer Pattern
- **Routes/Controllers:** HTTP layer (validation, headers, status)
- **Services:** Business logic (core functionality)
- **Utils/Helpers:** Pure functions (parsing, formatting)
- **Models:** Data structures and validation

---

## 5. Response Format

### Standard Pattern
1. **Think in Indonesian:** Briefly outline plan
2. **Implement:** Provide code with all imports
3. **Verify:** Run checks and report results

**Example:**
```
"Saya akan buat 3 file: routes untuk endpoints, update schemas, register di app."

[code block with imports]

✅ Syntax check passed. All files OK.
```

---

# Part B: Frontend Development

## Priority: Deployment Excellence

### Technology Stack
- **Markup:** HTML5 (semantic)
- **Styling:** CSS3, Tailwind CSS (optional)
- **Logic:** JavaScript (ES6+), TypeScript (preferred)
- **Frameworks:** React, Next.js, Vite, Astro
- **Deployment:** Vercel, Netlify, Cloudflare Pages

---

## 1. Project Structure

### Standard Organization
```
project/
├── src/
│   ├── components/      # Reusable UI components
│   ├── pages/           # Route pages
│   ├── styles/          # Global styles
│   ├── utils/           # Helper functions
│   ├── hooks/           # Custom React hooks (if React)
│   ├── types/           # TypeScript types
│   └── config/          # Configuration
├── public/              # Static assets
├── tests/               # Test files
└── package.json
```

### Component Structure (React/Next.js)
```typescript
// components/Button/Button.tsx
import { ButtonHTMLAttributes } from 'react';
import styles from './Button.module.css';

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary';
  loading?: boolean;
}

export const Button = ({ 
  variant = 'primary', 
  loading = false,
  children,
  ...props 
}: ButtonProps) => {
  return (
    <button 
      className={`${styles.button} ${styles[variant]}`}
      disabled={loading}
      {...props}
    >
      {loading ? 'Loading...' : children}
    </button>
  );
};
```

---

## 2. TypeScript Best Practices

### Strict Mode Always
```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

### Type Everything
```typescript
// ❌ Bad
const fetchData = async (id) => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
};

// ✅ Good
interface User {
  id: string;
  name: string;
  email: string;
}

const fetchData = async (id: string): Promise<User> => {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  return response.json();
};
```

---

## 3. CSS/Styling Standards

### Naming Convention: BEM or CSS Modules
```css
/* BEM */
.button { }
.button--primary { }
.button__icon { }

/* CSS Modules (preferred with React) */
.button { }
.primary { }
.icon { }
```

### Responsive Design (Mobile-First)
```css
/* Base (mobile) */
.container {
  padding: 1rem;
}

/* Tablet */
@media (min-width: 768px) {
  .container {
    padding: 2rem;
  }
}

/* Desktop */
@media (min-width: 1024px) {
  .container {
    padding: 3rem;
  }
}
```

---

## 4. Deployment Excellence (PRIORITY)

### Build Optimization
```json
// package.json
{
  "scripts": {
    "build": "vite build",
    "preview": "vite preview",
    "analyze": "vite-bundle-visualizer"
  }
}
```

### Code Splitting & Lazy Loading
```typescript
// React lazy loading
import { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <HeavyComponent />
    </Suspense>
  );
}
```

### Image Optimization
```typescript
// Next.js Image
import Image from 'next/image';

<Image
  src="/hero.jpg"
  alt="Hero image"
  width={1200}
  height={600}
  priority={true}  // For above-the-fold images
  placeholder="blur"
/>
```

### Environment Variables
```bash
# .env.local (never commit)
NEXT_PUBLIC_API_URL=https://api.example.com
DATABASE_URL=postgresql://...

# .env.example (commit this)
NEXT_PUBLIC_API_URL=
DATABASE_URL=
```

### Vercel Deployment Checklist
- ✅ `vercel.json` configured
- ✅ Environment variables set in Vercel dashboard
- ✅ Build command correct
- ✅ Output directory specified
- ✅ Serverless function size <50MB
- ✅ No hardcoded secrets
- ✅ CORS configured properly

**vercel.json Example:**
```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "framework": "vite",
  "rewrites": [
    { "source": "/api/(.*)", "destination": "/api" }
  ]
}
```

---

## 5. Testing (Frontend)

### Unit Tests (Jest/Vitest)
```typescript
import { render, screen } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  it('renders with text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('shows loading state', () => {
    render(<Button loading>Click me</Button>);
    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });
});
```

### E2E Tests (Playwright)
```typescript
import { test, expect } from '@playwright/test';

test('user can login', async ({ page }) => {
  await page.goto('http://localhost:3000');
  await page.fill('[name="email"]', 'user@example.com');
  await page.fill('[name="password"]', 'password123');
  await page.click('button[type="submit"]');
  
  await expect(page).toHaveURL('/dashboard');
});
```

---

## 6. Performance & SEO

### Meta Tags
```html
<!-- Essential meta tags -->
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="Your site description">
  <title>Your Page Title</title>
  
  <!-- Open Graph -->
  <meta property="og:title" content="Your Page Title">
  <meta property="og:description" content="Description">
  <meta property="og:image" content="/og-image.jpg">
</head>
```

### Core Web Vitals
- **LCP** (Largest Contentful Paint): <2.5s
- **FID** (First Input Delay): <100ms
- **CLS** (Cumulative Layout Shift): <0.1

**Monitoring:**
```typescript
// web-vitals package
import { getCLS, getFID, getLCP } from 'web-vitals';

getCLS(console.log);
getFID(console.log);
getLCP(console.log);
```

---

# Part C: Backend Development

## Priority: API Design Patterns Excellence

### Supported Languages
- **Python** (FastAPI, Django, Flask)
- **Go** (Gin, Echo, Fiber)
- **PHP** (Laravel, Symfony)

---

## 1. API Design Patterns (PRIORITY)

### RESTful API Standards

#### HTTP Methods
- `GET` — Read/retrieve resources
- `POST` — Create new resources
- `PUT/PATCH` — Update existing resources
- `DELETE` — Remove resources

#### Status Codes
- `200 OK` — Success
- `201 Created` — Resource created
- `400 Bad Request` — Invalid input
- `401 Unauthorized` — Missing/invalid auth
- `403 Forbidden` — No permission
- `404 Not Found` — Resource doesn't exist
- `500 Internal Server Error` — Server error

#### Resource Naming
```
✅ Good:
GET    /api/users           # List users
GET    /api/users/{id}      # Get specific user
POST   /api/users           # Create user
PUT    /api/users/{id}      # Update user
DELETE /api/users/{id}      # Delete user

❌ Bad:
GET    /api/getUsers
POST   /api/createUser
GET    /api/user-detail/{id}
```

### Request Validation

**Python (FastAPI/Pydantic):**
```python
from pydantic import BaseModel, Field, validator

class UserCreate(BaseModel):
    email: str = Field(..., regex=r"^[\w\.-]+@[\w\.-]+\.\w+$")
    password: str = Field(..., min_length=8)
    age: int = Field(..., ge=18, le=120)
    
    @validator('password')
    def password_strength(cls, v):
        if not any(c.isupper() for c in v):
            raise ValueError('Password must contain uppercase')
        return v
```

**Go (gin-validator):**
```go
type UserCreate struct {
    Email    string `json:"email" binding:"required,email"`
    Password string `json:"password" binding:"required,min=8"`
    Age      int    `json:"age" binding:"required,gte=18,lte=120"`
}

func CreateUser(c *gin.Context) {
    var user UserCreate
    if err := c.ShouldBindJSON(&user); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
    // Process user...
}
```

**PHP (Laravel):**
```php
use Illuminate\Http\Request;

public function store(Request $request)
{
    $validated = $request->validate([
        'email' => 'required|email|unique:users',
        'password' => 'required|min:8|regex:/[A-Z]/',
        'age' => 'required|integer|min:18|max:120',
    ]);
    
    // Process user...
}
```

### Response Format Consistency

**Success Response:**
```json
{
  "status": "success",
  "data": {
    "id": "123",
    "name": "John Doe"
  },
  "meta": {
    "timestamp": "2026-02-11T14:00:00Z",
    "request_id": "abc123"
  }
}
```

**Error Response:**
```json
{
  "status": "error",
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Email is required"
      }
    ]
  },
  "meta": {
    "timestamp": "2026-02-11T14:00:00Z",
    "request_id": "abc123"
  }
}
```

### Pagination

**Cursor-based (Recommended for large datasets):**
```python
@router.get("/users")
async def list_users(
    cursor: str | None = None,
    limit: int = 20
):
    # Fetch users after cursor
    users = await db.fetch_users_after(cursor, limit)
    
    return {
        "data": users,
        "pagination": {
            "next_cursor": users[-1].id if users else None,
            "has_more": len(users) == limit
        }
    }
```

**Offset-based (Simple, for small datasets):**
```python
@router.get("/users")
async def list_users(
    page: int = 1,
    per_page: int = 20
):
    offset = (page - 1) * per_page
    users = await db.fetch_users(offset, per_page)
    total = await db.count_users()
    
    return {
        "data": users,
        "pagination": {
            "page": page,
            "per_page": per_page,
            "total": total,
            "total_pages": (total + per_page - 1) // per_page
        }
    }
```

### API Versioning

**URL-based (Recommended):**
```
/api/v1/users
/api/v2/users
```

**Header-based:**
```
GET /api/users
Accept: application/vnd.myapi.v2+json
```

---

## 2. Error Handling Patterns

### Python (FastAPI)
```python
from fastapi import HTTPException, status
from typing import Optional

class APIError(Exception):
    def __init__(
        self,
        status_code: int,
        message: str,
        details: Optional[dict] = None
    ):
        self.status_code = status_code
        self.message = message
        self.details = details

@app.exception_handler(APIError)
async def api_error_handler(request: Request, exc: APIError):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "status": "error",
            "error": {
                "message": exc.message,
                "details": exc.details
            }
        }
    )
```

### Go (Gin)
```go
type APIError struct {
    StatusCode int         `json:"-"`
    Message    string      `json:"message"`
    Details    interface{} `json:"details,omitempty"`
}

func ErrorHandler() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Next()
        
        if len(c.Errors) > 0 {
            err := c.Errors[0].Err
            if apiErr, ok := err.(APIError); ok {
                c.JSON(apiErr.StatusCode, gin.H{
                    "status": "error",
                    "error": apiErr,
                })
                return
            }
            
            c.JSON(500, gin.H{
                "status": "error",
                "error": gin.H{"message": err.Error()},
            })
        }
    }
}
```

### PHP (Laravel)
```php
namespace App\Exceptions;

use Exception;

class APIException extends Exception
{
    protected $details;
    
    public function __construct($message, $statusCode = 400, $details = null)
    {
        parent::__construct($message, $statusCode);
        $this->details = $details;
    }
    
    public function render($request)
    {
        return response()->json([
            'status' => 'error',
            'error' => [
                'message' => $this->getMessage(),
                'details' => $this->details,
            ]
        ], $this->getCode());
    }
}
```

---

## 3. Database Best Practices

### Connection Pooling

**Python (asyncpg):**
```python
import asyncpg

pool = await asyncpg.create_pool(
    dsn="postgresql://user:pass@localhost/db",
    min_size=10,
    max_size=20,
    command_timeout=60
)

async with pool.acquire() as conn:
    result = await conn.fetch("SELECT * FROM users")
```

**Go (pgx):**
```go
import "github.com/jackc/pgx/v5/pgxpool"

pool, err := pgxpool.New(context.Background(),
    "postgres://user:pass@localhost/db?pool_max_conns=20")

rows, err := pool.Query(context.Background(),
    "SELECT * FROM users")
```

### Query Optimization
```python
# ❌ Bad: N+1 query problem
users = await db.fetch("SELECT * FROM users")
for user in users:
    posts = await db.fetch("SELECT * FROM posts WHERE user_id = $1", user.id)

# ✅ Good: Join or batch query
users_with_posts = await db.fetch("""
    SELECT u.*, json_agg(p.*) as posts
    FROM users u
    LEFT JOIN posts p ON p.user_id = u.id
    GROUP BY u.id
""")
```

### Transactions
```python
async with pool.acquire() as conn:
    async with conn.transaction():
        await conn.execute("INSERT INTO users (...) VALUES (...)")
        await conn.execute("INSERT INTO profiles (...) VALUES (...)")
        # Both succeed or both rollback
```

---

## 4. Authentication & Authorization

### JWT Pattern (Recommended)
```python
from jose import JWTError, jwt
from datetime import datetime, timedelta

SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"

def create_access_token(data: dict, expires_delta: timedelta):
    to_encode = data.copy()
    expire = datetime.utcnow() + expires_delta
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def verify_token(token: str):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except JWTError:
        raise HTTPException(401, "Invalid token")
```

### API Key Pattern
```python
from fastapi import Header, HTTPException

async def verify_api_key(x_api_key: str = Header(...)):
    if x_api_key != settings.API_KEY:
        raise HTTPException(403, "Invalid API key")
    return x_api_key
```

### Role-Based Access Control (RBAC)
```python
from enum import Enum

class Role(str, Enum):
    ADMIN = "admin"
    USER = "user"
    GUEST = "guest"

def require_role(required_role: Role):
    def decorator(func):
        async def wrapper(*args, **kwargs):
            current_user = get_current_user()
            if current_user.role != required_role:
                raise HTTPException(403, "Insufficient permissions")
            return await func(*args, **kwargs)
        return wrapper
    return decorator

@router.delete("/users/{id}")
@require_role(Role.ADMIN)
async def delete_user(id: str):
    # Only admins can delete users
    pass
```

---

## 5. Async Best Practices

### Python (FastAPI)
```python
# ✅ Good: Use async for I/O operations
@router.get("/users/{id}")
async def get_user(id: str):
    async with httpx.AsyncClient() as client:
        user = await client.get(f"https://api.example.com/users/{id}")
    return user.json()

# ❌ Bad: Blocking I/O in async function
@router.get("/users/{id}")
async def get_user(id: str):
    import requests  # Blocking!
    user = requests.get(f"https://api.example.com/users/{id}")
    return user.json()
```

### Go (Goroutines)
```go
func FetchMultipleUsers(ids []string) ([]User, error) {
    results := make(chan User, len(ids))
    errors := make(chan error, len(ids))
    
    var wg sync.WaitGroup
    for _, id := range ids {
        wg.Add(1)
        go func(id string) {
            defer wg.Done()
            user, err := fetchUser(id)
            if err != nil {
                errors <- err
                return
            }
            results <- user
        }(id)
    }
    
    wg.Wait()
    close(results)
    close(errors)
    
    // Collect results...
}
```

---

## 6. Logging & Monitoring

### Structured Logging

**Python:**
```python
import structlog

log = structlog.get_logger()

log.info(
    "user_login",
    user_id="123",
    ip_address="192.168.1.1",
    success=True
)
```

**Go:**
```go
import "go.uber.org/zap"

logger, _ := zap.NewProduction()
logger.Info("user_login",
    zap.String("user_id", "123"),
    zap.String("ip_address", "192.168.1.1"),
    zap.Bool("success", true),
)
```

### Request Logging Middleware
```python
import time
from fastapi import Request

@app.middleware("http")
async def log_requests(request: Request, call_next):
    start_time = time.time()
    
    log.info("request_started",
        method=request.method,
        path=request.url.path
    )
    
    response = await call_next(request)
    
    duration = time.time() - start_time
    log.info("request_completed",
        method=request.method,
        path=request.url.path,
        status_code=response.status_code,
        duration=duration
    )
    
    return response
```

---

## 7. Testing (Backend)

### Unit Tests
**Python (pytest):**
```python
import pytest
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_create_user():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.post(
            "/api/users",
            json={"email": "test@example.com", "password": "Test123"}
        )
        assert response.status_code == 201
        assert response.json()["data"]["email"] == "test@example.com"

@pytest.mark.asyncio
async def test_create_user_invalid_email():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.post(
            "/api/users",
            json={"email": "invalid", "password": "Test123"}
        )
        assert response.status_code == 400
```

### Integration Tests
```python
@pytest.mark.asyncio
async def test_user_flow():
    async with AsyncClient(app=app, base_url="http://test") as client:
        # Create user
        create_response = await client.post("/api/users", json=user_data)
        assert create_response.status_code == 201
        user_id = create_response.json()["data"]["id"]
        
        # Login
        login_response = await client.post("/api/login", json=credentials)
        token = login_response.json()["token"]
        
        # Fetch user with token
        user_response = await client.get(
            f"/api/users/{user_id}",
            headers={"Authorization": f"Bearer {token}"}
        )
        assert user_response.status_code == 200
```

---

## Summary: Key Principles

### Universal (All Languages)
1. **Verify Always** — Syntax/lint check after every change
2. **Test Everything** — Minimum 3 test cases per feature
3. **Communicate** — Ask for complex tasks, proceed for simple fixes
4. **Document** — Update README, inline comments
5. **Clean Code** — DRY, reuse, remove dead code

### Frontend Priority: Deployment
6. **Optimize Builds** — Code splitting, lazy loading, image optimization
7. **Environment Variables** — Never hardcode secrets
8. **Performance** — Monitor Core Web Vitals
9. **SEO** — Meta tags, semantic HTML
10. **Vercel Ready** — <50MB, proper configuration

### Backend Priority: API Design
11. **RESTful Patterns** — Consistent naming, methods, status codes
12. **Validation** — Validate all inputs with proper error messages
13. **Error Handling** — Structured errors, never silent failures
14. **Authentication** — JWT/API keys with proper RBAC
15. **Async Operations** — Non-blocking I/O for all DB/API calls

**Remember:** Quality > Speed. Verify, test, document properly before completing tasks.

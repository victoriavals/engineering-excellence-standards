# Backend API Design Patterns (Python/Go/PHP)

**Priority Focus: API Design Excellence**

---

## 1. REST API Design Principles

### URL Structure
```
✅ Good RESTful URLs:
GET    /api/users              # List users
GET    /api/users/{id}         # Get specific user
POST   /api/users              # Create user
PUT    /api/users/{id}         # Update user (full)
PATCH  /api/users/{id}         # Update user (partial)
DELETE /api/users/{id}         # Delete user

GET    /api/users/{id}/posts   # Get user's posts (nested resource)
POST   /api/users/{id}/posts   # Create post for user

❌ Avoid:
/api/getUsers
/api/user_list
/api/deleteUserById
```

### HTTP Status Codes (Correct Usage)
```
Success:
200 OK              # GET, PATCH successful
201 Created         # POST successful (include Location header)
204 No Content      # DELETE successful

Client Errors:
400 Bad Request     # Invalid input/validation error
401 Unauthorized    # Missing/invalid authentication
403 Forbidden       # Authenticated but no permission
404 Not Found       # Resource doesn't exist
409 Conflict        # Resource already exists
422 Unprocessable   # Validation errors (detailed)

Server Errors:
500 Internal Error  # Unexpected server error
502 Bad Gateway     # Upstream service failed
503 Service Unavailable  # Temporarily down
```

### Request/Response Format

**✅ Good API Response Structure:**
```json
{
  "status": "success",
  "data": {
    "id": "123",
    "name": "John Doe",
    "email": "john@example.com"
  },
  "meta": {
    "timestamp": "2026-02-11T13:00:00Z",
    "version": "v1"
  }
}
```

**✅ Good Error Response:**
```json
{
  "status": "error",
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      },
      {
        "field": "age",
        "message": "Must be at least 18"
      }
    ]
  },
  "meta": {
    "timestamp": "2026-02-11T13:00:00Z",
    "request_id": "req_abc123"
  }
}
```

---

## 2. API Versioning Strategies

### URL Versioning (Recommended for simplicity)
```
/api/v1/users
/api/v2/users  # Breaking changes in v2
```

### Header Versioning (Advanced)
```http
GET /api/users
Accept: application/vnd.myapi.v1+json
```

### Best Practices:
* Use semantic versioning (v1, v2, v3)
* Support at least 2 versions simultaneously
* Document deprecation timeline
* Provide migration guides

---

## 3. Authentication & Authorization

### JWT Token Pattern (Recommended)
```python
# Python (FastAPI)
from fastapi import Header, HTTPException
from jose import jwt, JWTError

def verify_token(authorization: str = Header(None)) -> dict:
    """Verify JWT token from Authorization header."""
    if not authorization or not authorization.startswith("Bearer "):
        raise HTTPException(401, "Missing or invalid token")
    
    token = authorization.replace("Bearer ", "")
    
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        return payload
    except JWTError:
        raise HTTPException(401, "Invalid token")
```

```go
// Go (using gin)
func AuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        authHeader := c.GetHeader("Authorization")
        if authHeader == "" {
            c.JSON(401, gin.H{"error": "Missing token"})
            c.Abort()
            return
        }
        
        token := strings.Replace(authHeader, "Bearer ", "", 1)
        claims, err := ValidateJWT(token)
        if err != nil {
            c.JSON(401, gin.H{"error": "Invalid token"})
            c.Abort()
            return
        }
        
        c.Set("user_id", claims.UserID)
        c.Next()
    }
}
```

```php
// PHP (using Laravel)
class AuthMiddleware
{
    public function handle($request, Closure $next)
    {
        $token = $request->bearerToken();
        
        if (!$token) {
            return response()->json([
                'error' => 'Missing token'
            ], 401);
        }
        
        try {
            $payload = JWT::decode($token, env('JWT_SECRET'), ['HS256']);
            $request->user_id = $payload->user_id;
        } catch (\Exception $e) {
            return response()->json([
                'error' => 'Invalid token'
            ], 401);
        }
        
        return $next($request);
    }
}
```

### API Key Pattern (Simpler alternative)
```http
GET /api/users
X-API-Key: your_secret_key_here
```

**When to use:**
* JWT: User authentication, sessions, role-based access
* API Key: Service-to-service, webhooks, public APIs

---

## 4. Input Validation & Sanitization

### Validate Everything
```python
# Python (Pydantic)
from pydantic import BaseModel, Field, validator

class CreateUserRequest(BaseModel):
    email: str = Field(..., min_length=5, max_length=100)
    age: int = Field(..., ge=18, le=120)
    name: str = Field(..., min_length=1, max_length=100)
    
    @validator('email')
    def validate_email(cls, v):
        if '@' not in v or '.' not in v:
            raise ValueError('Invalid email format')
        return v.lower()
```

```go
// Go (using validator)
type CreateUserRequest struct {
    Email string `json:"email" binding:"required,email,min=5,max=100"`
    Age   int    `json:"age" binding:"required,gte=18,lte=120"`
    Name  string `json:"name" binding:"required,min=1,max=100"`
}

func CreateUser(c *gin.Context) {
    var req CreateUserRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
    // Process request...
}
```

```php
// PHP (Laravel Request Validation)
class CreateUserRequest extends FormRequest
{
    public function rules()
    {
        return [
            'email' => 'required|email|min:5|max:100',
            'age' => 'required|integer|min:18|max:120',
            'name' => 'required|string|min:1|max:100',
        ];
    }
}
```

### Sanitization Rules:
* **Never trust user input**
* Strip HTML tags (unless intentional)
* Escape SQL queries (use parameterized queries)
* Validate file uploads (type, size, content)

---

## 5. Error Handling Standards

### Centralized Error Handler
```python
# Python (FastAPI)
from fastapi import Request
from fastapi.responses import JSONResponse

class APIException(Exception):
    def __init__(self, status_code: int, code: str, message: str):
        self.status_code = status_code
        self.code = code
        self.message = message

@app.exception_handler(APIException)
async def api_exception_handler(request: Request, exc: APIException):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "status": "error",
            "error": {
                "code": exc.code,
                "message": exc.message
            }
        }
    )
```

```go
// Go (Custom error middleware)
type APIError struct {
    StatusCode int    `json:"-"`
    Code       string `json:"code"`
    Message    string `json:"message"`
}

func ErrorHandler(c *gin.Context) {
    c.Next()
    
    if len(c.Errors) > 0 {
        err := c.Errors.Last().Err
        if apiErr, ok := err.(*APIError); ok {
            c.JSON(apiErr.StatusCode, gin.H{
                "status": "error",
                "error": gin.H{
                    "code":    apiErr.Code,
                    "message": apiErr.Message,
                },
            })
        } else {
            c.JSON(500, gin.H{
                "status": "error",
                "error": gin.H{
                    "code":    "INTERNAL_ERROR",
                    "message": "An unexpected error occurred",
                },
            })
        }
    }
}
```

### Error Response Consistency
* Always include `status: "error"`
* Always include `error.code` (machine-readable)
* Always include `error.message` (human-readable)
* Include `error.details` for validation errors
* Include `meta.request_id` for debugging

---

## 6. Database Best Practices

### Query Optimization
```python
# ❌ Bad: N+1 query problem
users = db.query(User).all()
for user in users:
    posts = db.query(Post).filter(Post.user_id == user.id).all()

# ✅ Good: Join/eager loading
users = db.query(User).options(joinedload(User.posts)).all()
```

### Connection Pooling
```python
# Python (SQLAlchemy)
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool

engine = create_engine(
    DATABASE_URL,
    poolclass=QueuePool,
    pool_size=10,        # Maintain 10 connections
    max_overflow=20,     # Allow 20 extra if needed
    pool_timeout=30,     # Wait 30s for connection
    pool_recycle=3600    # Recycle connections after 1hr
)
```

```go
// Go (database/sql)
db, err := sql.Open("postgres", connStr)
if err != nil {
    log.Fatal(err)
}

db.SetMaxOpenConns(25)           // Max open connections
db.SetMaxIdleConns(10)           // Max idle connections
db.SetConnMaxLifetime(time.Hour) // Max connection lifetime
```

### Transaction Management
```python
# Python (SQLAlchemy)
from sqlalchemy.orm import Session

def create_user_with_profile(db: Session, user_data: dict, profile_data: dict):
    try:
        user = User(**user_data)
        db.add(user)
        db.flush()  # Get user.id without committing
        
        profile = Profile(**profile_data, user_id=user.id)
        db.add(profile)
        
        db.commit()  # Commit both together
        return user
    except Exception as e:
        db.rollback()  # Rollback on error
        raise
```

---

## 7. Pagination & Filtering

### Cursor-Based Pagination (Recommended for large datasets)
```
GET /api/users?limit=20&cursor=abc123

Response:
{
  "data": [...],
  "pagination": {
    "next_cursor": "def456",
    "has_more": true
  }
}
```

### Offset-Based Pagination (Simpler, for small datasets)
```
GET /api/users?page=2&per_page=20

Response:
{
  "data": [...],
  "pagination": {
    "page": 2,
    "per_page": 20,
    "total": 150,
    "total_pages": 8
  }
}
```

### Filtering & Sorting
```
GET /api/users?status=active&role=admin&sort=-created_at

Rules:
- Use query parameters for filters
- Use `-` prefix for descending sort
- Support multiple filters with AND logic
- Document all filterable fields
```

---

## 8. Rate Limiting & Throttling

### Implementation Example (Python)
```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@app.get("/api/users")
@limiter.limit("100/minute")  # 100 requests per minute
async def list_users():
    return {"users": [...]}
```

### Headers to Return
```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 73
X-RateLimit-Reset: 1676123400
```

### Typical Limits:
* Public endpoints: 60 requests/minute
* Authenticated endpoints: 1000 requests/minute
* Write operations: 100 requests/minute

---

## 9. API Documentation

### OpenAPI/Swagger (Auto-generated)
```python
# Python (FastAPI auto-generates)
@app.post("/api/users", response_model=UserResponse, status_code=201)
async def create_user(user: CreateUserRequest):
    """
    Create a new user.
    
    - **email**: User's email address (unique)
    - **name**: User's full name
    - **age**: User's age (must be 18+)
    """
    # Create user logic...
```

### Documentation Requirements:
* Every endpoint must have:
    - Description/purpose
    - Request parameters (path, query, body)
    - Response format (success cases)
    - Error responses (all possible codes)
    - Example request/response
    - Authentication requirements

---

## 10. Testing Strategy

### API Test Pyramid
```
Unit Tests (60%)        # Business logic, utilities
Integration Tests (30%) # API endpoints + database
E2E Tests (10%)        # Full user flows
```

### Example Integration Test
```python
# Python (pytest + httpx)
import pytest
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_create_user():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.post(
            "/api/users",
            json={"email": "test@example.com", "name": "Test", "age": 25},
            headers={"X-API-Key": API_KEY}
        )
        
        assert response.status_code == 201
        data = response.json()
        assert data["status"] == "success"
        assert "id" in data["data"]
```

### Test Coverage Requirements:
* All endpoints: 90%+ coverage
* Critical paths (auth, payments): 100% coverage
* Error scenarios: Must be tested

---

## Summary: Backend API Best Practices

1. **RESTful Design** — Proper URLs, HTTP methods, status codes
2. **Consistent Responses** — Standard structure for success/error
3. **Version APIs** — Support backwards compatibility
4. **Secure By Default** — JWT/API keys, validate all inputs
5. **Handle Errors Gracefully** — Centralized handler, consistent format
6. **Optimize Queries** — Connection pooling, eager loading, indexes
7. **Paginate Everything** — Cursor or offset-based
8. **Rate Limit** — Protect from abuse
9. **Document Thoroughly** — OpenAPI/Swagger auto-generation
10. **Test Comprehensively** — 90%+ coverage, all error scenarios

**API Design Priority:** Every endpoint should be intuitive, secure, performant, and well-documented.

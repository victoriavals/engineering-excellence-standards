# Backend Development Rules (Go, PHP, Python)

**Extends:** [Universal Principles](./universal-principles.md)  
**Focus:** API Design Patterns, REST best practices, scalability

> **Note:** For Python-specific backend rules, see [python-backend.md](./python-backend.md)

---

## 1. API Design Patterns (CRITICAL)

### RESTful API Principles

#### Resource-Based URLs
```
✅ GOOD (Resource-oriented)
GET    /api/users              # List users
GET    /api/users/{id}         # Get specific user
POST   /api/users              # Create user
PUT    /api/users/{id}         # Update user (full)
PATCH  /api/users/{id}         # Update user (partial)
DELETE /api/users/{id}         # Delete user

❌ BAD (Action-oriented)
GET  /api/getAllUsers
POST /api/createNewUser
POST /api/deleteUserById
```

#### HTTP Status Codes (Correct Usage)
```
2xx Success
200 OK           → GET, PUT, PATCH success
201 Created      → POST success (return Location header)
204 No Content   → DELETE success

4xx Client Errors
400 Bad Request  → Validation error, malformed request
401 Unauthorized → Missing or invalid authentication
403 Forbidden    → Valid auth but no permission
404 Not Found    → Resource doesn't exist
422 Unprocessable Entity → Semantic errors (business logic)

5xx Server Errors
500 Internal Server Error → Unexpected server issues
503 Service Unavailable   → Temporary unavailability
```

#### Response Format (Consistent)
```json
// Success response
{
    "status": "success",
    "data": {
        "id": "user_123",
        "name": "John Doe",
        "email": "john@example.com"
    },
    "meta": {
        "timestamp": "2026-02-11T14:00:00Z",
        "request_id": "req_abc123"
    }
}

// Error response
{
    "status": "error",
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "Email is required",
        "details": [
            {
                "field": "email",
                "issue": "missing_required_field"
            }
        ]
    },
    "meta": {
        "timestamp": "2026-02-11T14:00:00Z",
        "request_id": "req_abc123"
    }
}
```

### API Versioning
```
# URL versioning (preferred for external APIs)
/api/v1/users
/api/v2/users

# Header versioning (for internal services)
Accept: application/vnd.myapp.v1+json
```

---

## 2. Go Backend Best Practices

### Project Structure
```
project/
├─ cmd/
│  └─ api/
│     └─ main.go           # Entry point
├─ internal/
│  ├─ handlers/            # HTTP handlers
│  ├─ services/            # Business logic
│  ├─ models/              # Data structures
│  └─ database/            # DB layer
├─ pkg/                    # Public libraries
├─ api/
│  └─ openapi.yaml         # API specification
└─ go.mod
```

### Handler Pattern (Gorilla Mux / Chi)
```go
package handlers

import (
    "encoding/json"
    "net/http"
    "github.com/gorilla/mux"
)

type User struct {
    ID    string `json:"id"`
    Name  string `json:"name" validate:"required"`
    Email string `json:"email" validate:"required,email"`
}

type Response struct {
    Status string      `json:"status"`
    Data   interface{} `json:"data,omitempty"`
    Error  *ErrorData  `json:"error,omitempty"`
}

type ErrorData struct {
    Code    string `json:"code"`
    Message string `json:"message"`
}

// GetUser retrieves a single user by ID
func GetUser(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    userID := vars["id"]
    
    // Business logic
    user, err := services.FetchUser(r.Context(), userID)
    if err != nil {
        respondError(w, http.StatusNotFound, "USER_NOT_FOUND", "User not found")
        return
    }
    
    respondJSON(w, http.StatusOK, Response{
        Status: "success",
        Data:   user,
    })
}

// Helper: JSON response
func respondJSON(w http.ResponseWriter, status int, payload interface{}) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(payload)
}

// Helper: Error response
func respondError(w http.ResponseWriter, status int, code, message string) {
    respondJSON(w, status, Response{
        Status: "error",
        Error: &ErrorData{
            Code:    code,
            Message: message,
        },
    })
}
```

### Middleware Pattern
```go
// Authentication middleware
func AuthMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        apiKey := r.Header.Get("X-API-Key")
        
        if apiKey == "" || !validateAPIKey(apiKey) {
            respondError(w, http.StatusUnauthorized, "INVALID_API_KEY", "Invalid API key")
            return
        }
        
        next.ServeHTTP(w, r)
    })
}

// Usage
r := mux.NewRouter()
r.Use(AuthMiddleware)
r.HandleFunc("/api/users/{id}", GetUser).Methods("GET")
```

### Error Handling
```go
// Custom error types
type AppError struct {
    Code    string
    Message string
    Err     error
}

func (e *AppError) Error() string {
    if e.Err != nil {
        return fmt.Sprintf("%s: %v", e.Message, e.Err)
    }
    return e.Message
}

// Service layer
func FetchUser(ctx context.Context, userID string) (*User, error) {
    user, err := db.QueryUser(ctx, userID)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, &AppError{
                Code:    "USER_NOT_FOUND",
                Message: "User not found",
                Err:     err,
            }
        }
        return nil, &AppError{
            Code:    "DATABASE_ERROR",
            Message: "Failed to fetch user",
            Err:     err,
        }
    }
    return user, nil
}
```

---

## 3. PHP Backend Best Practices

### Project Structure (Laravel/Symfony-style)
```
project/
├─ app/
│  ├─ Controllers/         # HTTP controllers
│  ├─ Services/            # Business logic
│  ├─ Models/              # Data models (Eloquent)
│  └─ Middleware/          # Request middleware
├─ routes/
│  └─ api.php              # API routes
├─ database/
│  ├─ migrations/
│  └─ seeders/
├─ config/
└─ composer.json
```

### Controller Pattern (Laravel)
```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Services\UserService;
use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;

class UserController extends Controller
{
    private UserService $userService;
    
    public function __construct(UserService $userService)
    {
        $this->userService = $userService;
    }
    
    /**
     * Get user by ID.
     *
     * @param string $id User ID
     * @return JsonResponse
     */
    public function show(string $id): JsonResponse
    {
        try {
            $user = $this->userService->findUser($id);
            
            return response()->json([
                'status' => 'success',
                'data' => $user,
            ], 200);
            
        } catch (\App\Exceptions\UserNotFoundException $e) {
            return response()->json([
                'status' => 'error',
                'error' => [
                    'code' => 'USER_NOT_FOUND',
                    'message' => $e->getMessage(),
                ],
            ], 404);
            
        } catch (\Exception $e) {
            return response()->json([
                'status' => 'error',
                'error' => [
                    'code' => 'INTERNAL_ERROR',
                    'message' => 'An error occurred',
                ],
            ], 500);
        }
    }
    
    /**
     * Create new user.
     *
     * @param Request $request
     * @return JsonResponse
     */
    public function store(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users',
        ]);
        
        $user = $this->userService->createUser($validated);
        
        return response()->json([
            'status' => 'success',
            'data' => $user,
        ], 201);
    }
}
```

### Service Layer (Dependency Injection)
```php
<?php

namespace App\Services;

use App\Models\User;
use App\Exceptions\UserNotFoundException;

class UserService
{
    /**
     * Find user by ID.
     *
     * @param string $id
     * @return User
     * @throws UserNotFoundException
     */
    public function findUser(string $id): User
    {
        $user = User::find($id);
        
        if (!$user) {
            throw new UserNotFoundException("User with ID {$id} not found");
        }
        
        return $user;
    }
    
    /**
     * Create new user.
     *
     * @param array $data
     * @return User
     */
    public function createUser(array $data): User
    {
        return User::create($data);
    }
}
```

### API Routes (routes/api.php)
```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\UserController;

Route::middleware('auth:api')->group(function () {
    Route::apiResource('users', UserController::class);
});

// Explicit routes (alternative)
Route::middleware('auth:api')->group(function () {
    Route::get('/users', [UserController::class, 'index']);
    Route::post('/users', [UserController::class, 'store']);
    Route::get('/users/{id}', [UserController::class, 'show']);
    Route::put('/users/{id}', [UserController::class, 'update']);
    Route::delete('/users/{id}', [UserController::class, 'destroy']);
});
```

---

## 4. Database Best Practices

### Connection Management
```go
// Go: Connection pooling
db, err := sql.Open("postgres", dsn)
db.SetMaxOpenConns(25)
db.SetMaxIdleConns(5)
db.SetConnMaxLifetime(5 * time.Minute)
```

```php
// PHP Laravel: config/database.php
'connections' => [
    'mysql' => [
        'driver' => 'mysql',
        'pool' => [
            'min' => 5,
            'max' => 25,
        ],
    ],
],
```

### Query Patterns (Avoid N+1)
```go
// ❌ BAD: N+1 queries
users := fetchAllUsers()
for _, user := range users {
    posts := fetchPostsByUserID(user.ID)  // N queries!
}

// ✅ GOOD: JOIN or eager loading
users := fetchUsersWithPosts()  // Single query with JOIN
```

```php
// ❌ BAD: N+1
$users = User::all();
foreach ($users as $user) {
    $posts = $user->posts;  // N queries!
}

// ✅ GOOD: Eager loading
$users = User::with('posts')->get();  // 2 queries total
```

---

## 5. Authentication & Authorization

### JWT Pattern
```go
// Go: JWT middleware
import "github.com/golang-jwt/jwt/v5"

type Claims struct {
    UserID string `json:"user_id"`
    jwt.RegisteredClaims
}

func generateToken(userID string) (string, error) {
    claims := Claims{
        UserID: userID,
        RegisteredClaims: jwt.RegisteredClaims{
            ExpiresAt: jwt.NewNumericDate(time.Now().Add(24 * time.Hour)),
        },
    }
    
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString([]byte(os.Getenv("JWT_SECRET")))
}
```

```php
// PHP Laravel: Sanctum/Passport
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens;
}

// Generate token
$token = $user->createToken('api-token')->plainTextToken;

// Protect routes
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

---

## 6. Rate Limiting & Caching

### Rate Limiting
```php
// Laravel: routes/api.php
Route::middleware('throttle:60,1')->group(function () {
    Route::apiResource('users', UserController::class);
});
```

```go
// Go: Rate limiting middleware
import "golang.org/x/time/rate"

func RateLimitMiddleware(limiter *rate.Limiter) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            if !limiter.Allow() {
                respondError(w, http.StatusTooManyRequests, "RATE_LIMIT", "Too many requests")
                return
            }
            next.ServeHTTP(w, r)
        })
    }
}
```

### Caching Strategies
```php
// Laravel: Cache API responses
use Illuminate\Support\Facades\Cache;

public function index()
{
    return Cache::remember('users_list', 3600, function () {
        return User::all();
    });
}
```

---

## 7. Testing

### API Testing (Go)
```go
func TestGetUser(t *testing.T) {
    req := httptest.NewRequest("GET", "/api/users/123", nil)
    req.Header.Set("X-API-Key", "test-key")
    
    w := httptest.NewRecorder()
    handler := http.HandlerFunc(GetUser)
    handler.ServeHTTP(w, req)
    
    assert.Equal(t, http.StatusOK, w.Code)
    
    var response Response
    json.Unmarshal(w.Body.Bytes(), &response)
    assert.Equal(t, "success", response.Status)
}
```

### API Testing (PHP Laravel)
```php
public function test_get_user_success()
{
    $user = User::factory()->create();
    
    $response = $this->getJson("/api/users/{$user->id}");
    
    $response->assertStatus(200)
             ->assertJson([
                 'status' => 'success',
                 'data' => [
                     'id' => $user->id,
                     'name' => $user->name,
                 ],
             ]);
}
```

---

## Summary: Backend API Checklist

✅ **RESTful Design** — Resource-based URLs, correct HTTP methods  
✅ **Consistent Responses** — Standardized JSON format (success/error)  
✅ **Proper Status Codes** — 2xx, 4xx, 5xx used correctly  
✅ **API Versioning** — /v1/, /v2/ for breaking changes  
✅ **Authentication** — JWT, OAuth2, API keys  
✅ **Authorization** — Role-based access control  
✅ **Rate Limiting** — Protect against abuse  
✅ **Error Handling** — Structured errors with codes  
✅ **Database Optimization** — Connection pooling, avoid N+1  
✅ **Caching** — Redis/Memcached for frequently accessed data  
✅ **API Documentation** — OpenAPI/Swagger spec  
✅ **Comprehensive Testing** — Unit + integration + API tests

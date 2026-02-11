---
description: Create new API endpoint/route with best practices
---

# New Endpoint Workflow

**Purpose:** Automate creation of new endpoint/route dengan boilerplate code, tests, dan documentation for ANY framework.

**Use Cases:**
- Adding new REST/GraphQL endpoint
- Rapid API development
- Ensure consistency across codebase
- Include tests from start

**Supported Frameworks:**
- **Python:** FastAPI, Django, Flask
- **JavaScript/TypeScript:** Express, NestJS, Next.js API routes
- **Go:** Gin, Echo, Fiber
- **PHP:** Laravel, Symfony
- **Ruby:** Rails

---

## Interactive Setup

### 1. Detect Framework & Gather Requirements

Ask user:

**Framework Detection:**
- "Apa framework yang kamu pakai?" (FastAPI/Express/Laravel/etc.)
- "Struktur project kamu seperti apa?" (routes/, controllers/, handlers/, etc.)

**Endpoint Details:**
1. **Endpoint name?** (e.g., "users", "products", "analytics")
2. **HTTP method?** (GET, POST, PUT, DELETE, PATCH)
3. **Need authentication?** (Yes/No, type: JWT/API Key/Session)
4. **Request schema?** (fields & types)
5. **Response schema?** (fields & types)
6. **Business logic location?** (inline/service layer/model)

---

### 2. Create Implementation Plan

Generate plan based on framework:

```markdown
#New Endpoint Implementation Plan

## Goal
Create [METHOD] /api/[endpoint] for [purpose]

## Files to Create/Modify

[Framework-specific structure]

### Python (FastAPI) Example:
- routes/[name]_routes.py (NEW)
- models/schemas.py (MODIFY - add Pydanticmodels)
- services/[name]_service.py (NEW - if using service layer)
- app.py (MODIFY - register router)
- test_[name].py (NEW)

### JavaScript (Express) Example:
- routes/[name].js (NEW)
- controllers/[name]Controller.js (NEW)
- models/[Name].js (NEW)
- app.js (MODIFY - register route)
- tests/[name].test.js (NEW)

### Go (Gin) Example:
- handlers/[name]_handler.go (NEW)
- models/[name].go (NEW)
- routes/routes.go (MODIFY)
- main.go (MODIFY)
- tests/[name]_test.go (NEW)
```

---

### 3. Generate Boilerplate Code

**AI will generate framework-specific code based on user's choice:**

#### Python (FastAPI) Template:
```python
# routes/[name]_routes.py
from fastapi import APIRouter, Depends

router = APIRouter(prefix="/[name]", tags=["[Name]"])

@router.get("/")
async def list_items():
    """List all items."""
    return {"items": []}

@router.post("/")
async def create_item(data: ItemCreate):
    """Create new item."""
    return {"id": "123", **data.dict()}
```

#### JavaScript (Express) Template:
```javascript
// routes/[name].js
const express = require('express');
const router = express.Router();

router.get('/', async (req, res) => {
  // List items
  res.json({ items: [] });
});

router.post('/', async (req, res) => {
  // Create item
  res.status(201).json({ id: '123', ...req.body });
});

module.exports = router;
```

#### Go (Gin) Template:
```go
// handlers/[name]_handler.go
package handlers

import "github.com/gin-gonic/gin"

func List[Name]s(c *gin.Context) {
    c.JSON(200, gin.H{"items": []interface{}{}})
}

func Create[Name](c *gin.Context) {
    var item Item
    if err := c.ShouldBindJSON(&item); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
    c.JSON(201, item)
}
```

---

### 4. Add Validation & Error Handling

Based on framework, add proper validation:

**FastAPI (Pydantic):**
```python
from pydantic import BaseModel, Field

class ItemCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    price: float = Field(..., gt=0)
```

**Express (express-validator):**
```javascript
const { body, validationResult } = require('express-validator');

router.post('/', [
  body('name').isLength({ min: 1, max: 100 }),
  body('price').isFloat({ gt: 0 })
], async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(422).json({ errors: errors.array() });
  }
  // ...
});
```

**Go (gin validator):**
```go
type Item struct {
    Name  string  `json:"name" binding:"required,min=1,max=100"`
    Price float64 `json:"price" binding:"required,gt=0"`
}
```

---

### 5. Add Authentication (if needed)

**FastAPI:**
```python
from fastapi import Depends, HTTPException
from fastapi.security import HTTPBearer

security = HTTPBearer()

@router.post("/", dependencies=[Depends(security)])
async def protected_endpoint():
    # ...
```

**Express:**
```javascript
const authMiddleware = require('../middleware/auth');

router.post('/', authMiddleware, async (req, res) => {
  // req.user available here
});
```

**Go:**
```go
func AuthMiddleware()gin.HandlerFunc {
    return func(c *gin.Context) {
        token := c.GetHeader("Authorization")
        // Validate token...
        c.Next()
    }
}

router.POST("/items", AuthMiddleware(), CreateItem)
```

---

### 6. Generate Test Cases

**Framework-specific test template:**

**Python (pytest):**
```python
import pytest
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_create_item():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.post("/api/items", 
            json={"name": "Test", "price": 10.0})
        assert response.status_code == 201
```

**JavaScript (Jest):**
```javascript
const request = require('supertest');
const app = require('../app');

describe('POST /api/items', () => {
  it('should create item', async () => {
    const res = await request(app)
      .post('/api/items')
      .send({ name: 'Test', price: 10.0 });
    expect(res.statusCode).toBe(201);
  });
});
```

**Go (testing):**
```go
func TestCreateItem(t *testing.T) {
    router := setupRouter()
    w := httptest.NewRecorder()
    
    body := `{"name":"Test","price":10.0}`
    req, _ := http.NewRequest("POST", "/api/items", strings.NewReader(body))
    router.ServeHTTP(w, req)
    
    assert.Equal(t, 201, w.Code)
}
```

---

### 7. Update Documentation

Add to README/API docs:

```markdown
### [METHOD] /api/[endpoint]

**Purpose:** [Description]

**Request:**
```json
{
  "field1": "value",
  "field2": 123
}
```

**Response ([STATUS]):**
```json
{
  "id": "uuid",
  "field1": "value",
  "created_at": "timestamp"
}
```

**Authentication:** [Required/Optional]
```

---

### 8. Register Route

**Framework-specific registration:**

**FastAPI:**
```python
# app.py
from routes.[name]_routes import router as [name]_router
app.include_router([name]_router, prefix="/api")
```

**Express:**
```javascript
// app.js
const [name]Routes = require('./routes/[name]');
app.use('/api/[name]', [name]Routes);
```

**Go:**
```go
// main.go
router.GET("/api/[name]", handlers.List[Name]s)
router.POST("/api/[name]", handlers.Create[Name])
```

---

### 9. Verify & Test

```bash
// turbo
# Syntax check
[language-specific syntax check command]

# Run tests
[language-specific test command]

# Start server and test manually
curl -X POST http://localhost:[PORT]/api/[endpoint] -d '{...}'
```

---

## Workflow Variants

- **CRUD Complete:** Generate all 5 methods (LIST, GET, POST, PUT, DELETE)
- **Simple:** No service layer, logic in routes
- **Complex:** Full service layer + repository pattern
- **GraphQL:** Generate resolver instead of REST endpoint

---

## Summary

This workflow adapts to ANY framework and generates:
- ✅ Route/handler boilerplate
- ✅ Request/response models
- ✅ Validation
- ✅ Authentication (if needed)
- ✅ Error handling
- ✅ Test cases
- ✅ Documentation

**Saves 30-45 minutes per endpoint with consistent quality across any tech stack!**

---
description: Test all API endpoints and services are responding
---

# API Health Check Workflow

**Purpose:** Verify API endpoints responding correctly with comprehensive health checks for ANY API framework.

**Use Cases:**
- Integration testing
- Post-deployment verification
- Debug API issues
- Monitor service availability

**Supported Frameworks:**
- REST APIs (FastAPI, Express, Django, Flask, Gin, Laravel, etc.)
- GraphQL APIs (Apollo, Hasura, etc.)
- gRPC services
- WebSocket servers

---

## Health Check Steps

### 1. Detect API Type & Port

Ask user:
- "Apa framework/type API yang kamu pakai?"
- "Port berapa? (default: 8000 untuk Python, 3000 untuk Node, 8080 untuk Go)"

---

### 2. Check if Server Running

```bash
// turbo
# Test connection to port
$port = [USER_SPECIFIED_PORT]
$url = "http://localhost:$port"

try {
    $response = Invoke-WebRequest -Uri "$url" -TimeoutSec 5 -ErrorAction Stop
    Write-Host "✅ Server is running on port $port"
} catch {
    Write-Host "❌ Server not responding on port $port"
    Write-Host "Start server first!"
}
```

---

### 3. Test Health/Status Endpoint

**Common health endpoint patterns:**
- `/health`
- `/api/health`
- `/status`
- `/ping`
- `/_health`

```bash
$healthEndpoints = @("/health", "/api/health", "/status", "/ping", "/_health")

foreach ($endpoint in $healthEndpoints) {
    try {
        $response = Invoke-WebRequest -Uri "$url$endpoint" -ErrorAction SilentlyContinue
        if ($response.StatusCode -eq 200) {
            Write-Host "✅ Health endpoint found: $endpoint"
            break
        }
    } catch {
        # Try next endpoint
    }
}
```

---

### 4. List Available Endpoints

**Framework-specific documentation endpoints:**

**OpenAPI/Swagger (REST):**
```bash
# Try common doc endpoints
curl http://localhost:$port/docs          # FastAPI, NestJS
curl http://localhost:$port/swagger       # Spring Boot, Express
curl http://localhost:$port/api-docs      # Generic
curl http://localhost:$port/openapi.json  # OpenAPI spec
```

**GraphQL:**
```bash
curl http://localhost:$port/graphql -d '{"query": "{__schema{types{name}}}"}' 
```

**Ask user:** "Mau test specific endpoints atau auto-discover dari docs?"

---

### 5. Test Core Endpoints

User provides list of endpoints to test, OR we extract from API docs.

**Generic endpoint test:**
```powershell
$endpoints = @(
    @{Method="GET"; Path="/api/users"; Auth=$false},
    @{Method="POST"; Path="/api/login"; Auth=$false; Body='{"username":"test"}'},
    @{Method="GET"; Path="/api/protected"; Auth=$true}
)

foreach ($ep in $endpoints) {
    $headers = @{"Content-Type" = "application/json"}
    
    # Add auth if needed
    if ($ep.Auth) {
        $headers["Authorization"] = "Bearer $env:API_TOKEN"
        # Or: $headers["X-API-Key"] = "$env:API_KEY"
    }
    
    $start = Get-Date
    try {
        if ($ep.Method -eq "GET") {
            $response = Invoke-WebRequest -Uri "$url$($ep.Path)" -Headers $headers
        } else {
            $response = Invoke-WebRequest -Uri "$url$($ep.Path)" -Method $ep.Method -Headers $headers -Body $ep.Body
        }
        
        $duration = ((Get-Date) - $start).TotalMilliseconds
        Write-Host "✅ $($ep.Method) $($ep.Path): $($response.StatusCode) (${duration}ms)"
    } catch {
        Write-Host "❌ $($ep.Method) $($ep.Path): Failed - $($_.Exception.Message)"
    }
}
```

---

### 6. Test Error Handling

```bash
# Test invalid auth
curl -X POST $url/api/protected # No auth header
# Expected: 401 Unauthorized

# Test invalid input
curl -X POST $url/api/users -d '{"invalid": "data"}'
# Expected: 400/422 Validation Error

# Test non-existent endpoint
curl $url/api/nonexistent
# Expected: 404 Not Found
```

---

### 7. Response Time Analysis

```powershell
$measurements = @()

1..10 | ForEach-Object {
    $start = Get-Date
    Invoke-WebRequest -Uri "$url/api/health" | Out-Null
    $duration = ((Get-Date) - $start).TotalMilliseconds
    $measurements += $duration
}

$avg = ($measurements | Measure-Object -Average).Average
$min = ($measurements | Measure-Object -Minimum).Minimum
$max = ($measurements | Measure-Object -Maximum).Maximum

Write-Host "Response Time Stats:"
Write-Host "  Average: ${avg}ms"
Write-Host "  Min: ${min}ms"
Write-Host "  Max: ${max}ms"
```

**Performance Rating:**
- < 100ms: 🟢 Excellent
- 100-500ms: 🟡 Good
- > 500ms: 🔴 Slow (investigate)

---

### 8. Generate Health Report

```markdown
# API Health Report

## Server Status
✅ Server running on port [PORT]
✅ Health endpoint responding
✅ Documentation available at [URL]

## Endpoint Status
| Endpoint | Method | Status | Response Time | Notes |
|---|---|---|---|---|
| /api/health | GET | ✅ 200 | 45ms | OK |
| /api/users | GET | ✅ 200 | 120ms | OK |
| /api/login | POST | ✅ 200 | 280ms | OK |
| /api/protected | GET | ✅ 401 | 50ms | Auth required (expected) |

## Error Handling
✅ Invalid auth properly rejected (401)
✅ Missing fields validation working (422)
✅ 404 errors properly formatted

## Performance
- Average response time: [XX]ms
- Slowest endpoint: [endpoint] ([XX]ms)
- All endpoints < 500ms: ✅

## Overall Status
🟢 ALL SYSTEMS OPERATIONAL

Issues Found: None
```

---

### 9. Cleanup (if applicable)

```bash
# Delete test data created during health check
# Adapt based on your API
```

---

## Framework-Specific Notes

**FastAPI:**
- Default port: 8000
- Docs: `/docs` (Swagger), `/redoc`
- Health: Custom endpoint

**Express/Node.js:**
- Default port: 3000
- Docs: Depends on setup (Swagger)
- Health: Custom endpoint

**Django Rest Framework:**
- Default port: 8000
- Docs: `/api/docs/` or `/swagger/`
- Health: Custom or `django-health-check`

**Spring Boot:**
- Default port: 8080
- Docs: `/swagger-ui.html`
- Health: `/actuator/health`

**Go (Gin/Echo/Fiber):**
- Default port: 8080
- Docs: Custom Swagger setup
- Health: Custom endpoint

---

## Quick One-Liner

```bash
# Quick health check
curl http://localhost:8000/health && echo "✅ API is healthy"

# Auto-monitor every 60s
while ($true) {
    $status = (Invoke-WebRequest -Uri "http://localhost:8000/health").StatusCode
    Write-Host "[$(Get-Date -Format 'HH:mm:ss')] Status: $status"
    Start-Sleep -Seconds 60
}
```

**This workflow works with ANY API framework or language!**

---
description: Scan codebase for security issues and vulnerabilities
---

# Security Scan Workflow

**Purpose:** Comprehensive security audit untuk detect hardcoded secrets, vulnerabilities, dan security misconfigurations.

**Use Cases:**
- Pre-deployment security check
- Regular security audits
- Compliance verification
- Find exposed secrets

---

## Security Checks

### 1. Scan for Hardcoded Secrets

// turbo

**Common Patterns:**
```powershell
# API Keys
Select-String -Path "*.py" -Pattern '(api_key|apikey)\s*=\s*["\'][^"\'get]+["\']' -CaseSensitive

# Passwords
Select-String -Path "*.py" -Pattern '(password|passwd|pwd)\s*=\s*["\'][^"\']+["\']' -CaseSensitive

# Tokens
Select-String -Path "*.py" -Pattern '(token|secret|key)\s*=\s*["\'][a-zA-Z0-9]{20,}["\']' -CaseSensitive

# Database URLs with credentials
Select-String -Path "*.py" -Pattern 'postgresql://.*:.*@' -CaseSensitive
```

**Report Findings:**
```
Found hardcoded secrets:
- file.py:42: api_key = "abc123..."
- config.py:15: password = "..."
```

**Recommended Fix:**
```python
# ❌ Bad
API_KEY = "sk-1234567890abcdef"

# ✅ Good
API_KEY = os.getenv("API_KEY")
```

---

### 2. Check .gitignore Configuration

Verify sensitive files are ignored:

```powershell
$requiredIgnores = @(".env", "*.log", "__pycache__", "*.pyc", ".pytest_cache")

$gitignoreContent = Get-Content ".gitignore"

foreach ($pattern in $requiredIgnores) {
    if ($gitignoreContent -notcontains $pattern) {
        Write-Host "⚠️ Missing in .gitignore: $pattern"
    } else {
        Write-Host "✅ Ignored: $pattern"
    }
}
```

---

### 3. Scan for SQL Injection Vulnerabilities

```powershell
# Raw SQL concatenation (vulnerable)
Select-String -Path "*.py" -Pattern 'execute\(.*\+.*\)' -CaseSensitive
Select-String -Path "*.py" -Pattern 'f"SELECT.*\{.*\}"' -CaseSensitive
```

**Flagged Patterns:**
```python
# ❌ Vulnerable to SQL injection
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
cursor.execute("SELECT * FROM users WHERE name = '" + name + "'")

# ✅ Safe (parameterized)
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
await conn.fetch("SELECT * FROM users WHERE id = $1", user_id)
```

---

### 4. Check for Exposed Debug Settings

```powershell
# Debug mode in production
Select-String -Path "*.py" -Pattern 'DEBUG\s*=\s*True' -CaseSensitive

# Exposed stack traces
Select-String -Path "*.py" -Pattern 'app\.run\(debug=True\)' -CaseSensitive
```

**Warning:**
```
❌ Found: DEBUG = True in config.py
   Risk: Exposes sensitive error information in production
   Fix: Use DEBUG = os.getenv("DEBUG", "False") == "True"
```

---

### 5. Dependency Vulnerability Scan

```bash
// turbo
# Python
pip install safety
safety check --json

# Node.js (if exists)
npm audit
```

**Report:**
```
Vulnerability Report:
- Package: urllib3
  Version: 1.26.0
  Issue: CVE-2021-xxxxx
  Recommendation: Upgrade to 1.26.5+
```

---

### 6. CORS Configuration Check

```python
# Scan for overly permissive CORS
Select-String -Path "*.py" -Pattern 'allow_origins=\["\*"\]' -CaseSensitive
```

**Warning:**
```
⚠️ CORS allows all origins (*)
   Risk: Potential CSRF attacks
   Recommendation: Specify allowed origins explicitly
```

**Better Config:**
```python
# ❌ Too permissive
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"]
)

# ✅ Restrictive
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://yourdomain.com",
        "https://app.yourdomain.com"
    ]
)
```

---

### 7. Authentication & Authorization Check

Verify endpoints have proper auth:

```powershell
# Find routes without authentication
Select-String -Path "routes/*.py" -Pattern '@router\.(get|post|put|delete)\(' -Context 0,5 | 
    Where-Object {$_.Context.PostContext -notmatch 'verify_api_key|Depends\(get_current_user\)'}
```

**Report:**
```
⚠️ Unprotected endpoints found:
- routes/users.py:42: @router.get("/users")
  Missing authentication check

Recommendation: Add @require_auth decorator or dependency
```

---

### 8. Environment Variable Validation

Check critical env vars are documented:

```powershell
# Get all os.getenv() calls
$envVars = Select-String -Path "*.py" -Pattern 'os\.getenv\(["\']([^"\']+)["\']' -AllMatches |
    ForEach-Object {$_.Matches.Groups[1].Value} |
    Sort-Object -Unique

# Check if in .env.example
$exampleVars = Get-Content ".env.example" | 
    Where-Object {$_ -match '^[A-Z_]+'} |
    ForEach-Object {($_ -split '=')[0]}

$missingDocs = Compare-Object $envVars $exampleVars -PassThru

if ($missingDocs) {
    Write-Host "⚠️ Undocumented environment variables:"
    $missingDocs | ForEach-Object {Write-Host "  -

 $_"}
}
```

---

### 9. Generate Security Report

```markdown
# Security Audit Report

## Critical Issues ❌
1. Hardcoded API key in config.py:15
2. SQL injection vulnerability in users.py:42
3. DEBUG=True in production config

## Warnings ⚠️
1. CORS allows all origins (*)
2. 3 unprotected endpoints found
3. Missing in .gitignore: .env

## Best Practices ✅
- All passwords hashed
- HTTPS enforced
- Rate limiting enabled
- Input validation present

## Dependency Vulnerabilities
- urllib3 (CVE-2021-xxxxx): Upgrade to 1.26.5+

## Recommendations
1. Move all secrets to environment variables
2. Use parameterized queries for SQL
3. Set DEBUG to False in production
4. Restrict CORS origins
5. Add authentication to exposed endpoints
6. Update vulnerable dependencies

## Overall Security Score
📊 6/10 - Medium Risk

**Action Required:** Fix critical issues before deployment
```

---

## Quick Security Checklist

```markdown
Before deploying:
- [ ] No hardcoded secrets
- [ ] .env not in git
- [ ] DEBUG = False
- [ ] CORS restricted
- [ ] All endpoints authenticated
- [ ] Dependencies up to date
- [ ] SQL queries parameterized
- [ ] Error messages don't expose internals
```

**Run `/security-scan` before every production deployment!**

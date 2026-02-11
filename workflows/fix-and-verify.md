---
description: Auto-fix common issues and verify code quality
---

# Fix and Verify Workflow

**Purpose:** Combine auto-fixing tools dengan comprehensive verification untuk ensure code quality before committing.

**Use Cases:**
- Before git commit
- After implementing feature
- Quick cleanup session
- Pre-PR checklist

---

// turbo-all

## Fix & Verify Steps

### 1. Auto-Fix Code Issues

**Run Formatters:**
```bash
# Black formatting
black . --line-length 100

# Import sorting
isort . --profile black

# Remove unused code
autoflake --in-place --remove-all-unused-imports --remove-unused-variables --recursive .
```

**Report:**
```
🔧 Auto-Fix Results:
- Files formatted: 8
- Imports reorganized: 5
- Unused code removed: 12 items
```

---

### 2. Syntax Verification

```bash
# Check all .py files
Get-ChildItem -Recurse -Filter "*.py" | ForEach-Object {
    python -c "import ast; ast.parse(open('$($_.FullName)', encoding='utf-8').read()); print('✅ $($_.Name): OK')"
}
```

**Stop if any errors found.**

---

### 3. Type Checking (mypy)

```bash
# Quick type check
mypy . --ignore-missing-imports --no-error-summary
```

**Report:**
```
Type Checking:
✅ No critical type errors
⚠️ 3 functions missing type hints (non-blocking)
```

---

### 4. Run Test Suite

```bash
# Run all tests
pytest tests/ -v --tb=short

# If no tests/ folder, run test scripts
Get-ChildItem -Filter "test_*.py" | ForEach-Object {
    python $_.Name
}
```

**Report:**
```
Tests:
✅ 42/42 passed (100%)
Time: 3.2s
```

---

### 5. Security Quick Scan

```bash
# Check for hardcoded secrets
$secrets = Select-String -Path "*.py" -Pattern '(api_key|password|token)\s*=\s*["\'][^"\'get]+["\']'

if ($secrets.Count -gt 0) {
    Write-Host "❌ Found hardcoded secrets:"
    $secrets | ForEach-Object {Write-Host "  $_"}
    # Ask to fix
} else {
    Write-Host "✅ No hardcoded secrets"
}
```

---

### 6. Verify Environment Variables

```bash
# All env vars documented?
$envVars = Select-String -Path "*.py" -Pattern 'os\.getenv\(["\']([^"\']+)["\']' -AllMatches |
    ForEach-Object {$_.Matches.Groups[1].Value} | Sort-Object -Unique

$exampleVars = Get-Content ".env.example" | 
    Where-Object {$_ -match '^[A-Z_]+'} |
    ForEach-Object {($_ -split '=')[0]}

$missing = $envVars | Where-Object {$exampleVars -notcontains $_}

if ($missing) {
    Write-Host "⚠️ Missing in .env.example:"
    $missing | ForEach-Object {Write-Host "  $_"}
} else {
    Write-Host "✅ All env vars documented"
}
```

---

### 7. Check Git Status

```bash
# What files changed?
git status --short

# Any untracked files that should be committed?
$untracked = git ls-files --others --exclude-standard

if ($untracked) {
    Write-Host "📋 Untracked files:"
    $untracked | ForEach-Object {Write-Host "  $_"}
    # Ask: "Add these to git?"
}
```

---

### 8. Generate Final Report

```markdown
# Fix & Verify Report

## ✅ Auto-Fixes Applied

### Code Formatting
- Files formatted: 8
- Imports sorted: 5 files
- Unused code removed: 12 items

## ✅ Verifications Passed

### Syntax Check
✅ All 15 Python files: OK

### Type Checking
✅ No critical errors
⚠️ 3 optional improvements (can be ignored)

### Tests
✅ 42/42 tests passed
⏱️ Execution time: 3.2s

### Security
✅ No hardcoded secrets
✅ .env properly ignored

### Configuration
✅ All env vars documented
✅ vercel.json present

### Git Status
- Modified files: 8
- Untracked files: 0
- Ready to commit: ✅

## Overall Status
🟢 ALL CHECKS PASSED

---

**You are safe to commit!** 🚀

Suggested commit command:
```bash
git add .
git commit -m "feat: [your description]"
git push
```
```

---

### 9. Interactive Commit (Optional)

Ask: "Mau commit sekarang?"

If yes:
```bash
# Show files to be committed
git status

# Ask for commit message
Read-Host "Enter commit message"

# Commit
git add .
git commit -m "$commitMessage"

# Ask to push
$push = Read-Host "Push to remote? (y/n)"
if ($push -eq "y") {
    git push
    Write-Host "✅ Changes pushed!"
}
```

---

## Quick Fix Phases

**Phase 1: Format (fast)**
- Black, isort, autoflake only
- ~10 seconds

**Phase 2: Verify (medium)**
- Phase 1 + Syntax + Tests
- ~30-60 seconds

**Phase 3: Complete (thorough)**
- All 8 steps
- ~2-3 minutes

---

## Error Handling

**If Syntax Errors:**
```
❌ Syntax error in routes/users.py:45
   Fix this before continuing.
   
Want AI to help fix? (y/n)
```

**If Tests Fail:**
```
❌ 3/42 tests failed

Failed tests:
- test_users.py::test_create_user
- test_auth.py::test_login
- test_memory.py::test_clear

Want to see failure details? (y/n)
```

**If Security Issues:**
```
❌ Security issue: Hardcoded API key in config.py:23

Auto-fix:
```python
# Before
API_KEY = "sk-1234567890"

# After
API_KEY = os.getenv("API_KEY")
```

Apply fix? (y/n)
```

---

## One-Liner

**Quick fix-and-verify:**
```powershell
black . && isort . --profile black && pytest tests/ -v && Write-Host "✅ Ready to commit!"
```

**This workflow saves time and prevents committing broken code!**

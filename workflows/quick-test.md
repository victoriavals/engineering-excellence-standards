---
description: Quick comprehensive test before commit
---

# Quick Test Workflow

**Purpose:** Run comprehensive pre-commit checks untuk ensure code quality before committing changes.

**Use Cases:**
- Before `git commit`
- Before creating PR
- Quick validation after multiple changes
- CI/CD preview

---

## Workflow Steps

### 1. Identify Modified Files

```bash
# Get modified Python files
git diff --name-only HEAD --diff-filter=ACMR | Select-String -Pattern '.py$'
```

**Output:** List of changed `.py` files

---

### 2. Syntax Check All Modified Python Files

// turbo
For each `.py` file:
```bash
python -c "import ast; ast.parse(open('file.py', encoding='utf-8').read()); print('✅ file.py: OK')"
```

**If Errors Found:**
- Stop workflow
- Show error details
- Ask: "Mau saya bantu fix syntax errors?"

---

### 3. Run Existing Test Scripts

// turbo  
Check for test files dalam project:

```bash
# Check for pytest
if (Test-Path "tests") {
    pytest tests/ -v
}

# Check for custom test scripts
if (Test-Path "test_*.py") {
    Get-ChildItem -Filter "test_*.py" | ForEach-Object {
        python $_.Name
    }
}
```

**Report:**
- Tests passed: [count]
- Tests failed: [count]
- Exit code: [0 = success]

---

### 4. Check Server Logs (if applicable)

If FastAPI/Flask app detected:

```bash
# Start server in background (if not running)
# Check for errors in startup

# Sample request to /api/health
curl http://localhost:8000/api/health
```

**Check for:**
- Server starts without errors
- Health endpoint responds
- No warnings in logs

---

### 5. Environment Variables Check

Verify `.env.example` is up to date:

```bash
# Get keys from .env
$envKeys = Get-Content .env | Where-Object {$_ -match '^[A-Z_]+'} | ForEach-Object {($_ -split '=')[0]}

# Get keys from .env.example
$exampleKeys = Get-Content .env.example | Where-Object {$_ -match '^[A-Z_]+'} | ForEach-Object {($_ -split '=')[0]}

# Compare
$missing = Compare-Object $envKeys $exampleKeys
```

**Report Missing Keys in .env.example**

---

### 6. Generate Final Report

```markdown
# Quick Test Report

## Syntax Checks
✅ All [N] modified files passed syntax check

## Test Results
✅ [N] tests passed
❌ [N] tests failed (if any)

## Server Status
✅ Server starts successfully
✅ Health endpoint responding

## Environment Variables
✅ .env.example up to date
⚠️ Missing in .env.example: [list] (if any)

## Overall Status
✅ READY TO COMMIT
```

or

```
❌ ISSUES FOUND - Fix before committing
```

---

## Auto-Run with // turbo-all

Add this annotation to auto-run all steps:

```markdown
// turbo-all
```

Then steps 2-5 will execute without asking for confirmation.

---

## Tips

**Fast Mode:** Only syntax check (Steps 1-2)
**Full Mode:** All 6 steps

**Recommended:** Run `/quick-test` before every commit to maintain code quality!

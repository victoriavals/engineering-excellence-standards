---
description: Auto-fix linting issues and format code (Python)
---

# Lint Fix Workflow (Universal - Python)

**Purpose:** Automatically fix common code quality issues and format Python code to standards. Works for ANY Python project.

**Use Cases:**
- Pre-commit cleanup
- Code formatting
- Import organization
- Style consistency

---

// turbo-all

## Auto-Fix Steps

### 1. Python Code Formatting (Black)

```bash
# Install black if needed
pip install black

# Format all Python files
black . --line-length 100

# Format specific directory
black src/ app/ lib/
```

**What Black Fixes:**
- Inconsistent indentation
- Line length (wraps to 100 chars)
- String quote consistency
- Whitespace normalization

---

### 2. Import Sorting (isort)

```bash
# Install isort
pip install isort

# Sort all imports
isort . --profile black

# Preview changes first
isort . --diff --check-only
```

**What isort Fixes:**
- Import order (stdlib → third-party → local)
- Grouped imports
- Alphabetical sorting
- Duplicate imports removed

---

### 3. Remove Unused Imports (autoflake)

```bash
# Install autoflake
pip install autoflake

# Remove unused imports
autoflake --in-place --remove-all-unused-imports --recursive .

# Also remove unused variables
autoflake --in-place --remove-all-unused-imports --remove-unused-variables --recursive .
```

---

### 4. Fix Common Issues (autopep8)

```bash
# Install autopep8
pip install autopep8

# Fix PEP 8 violations
autopep8 --in-place --aggressive --recursive .
```

**What autopep8 Fixes:**
- Whitespace issues
- Indentation
- Line continuations
- Comparison operators spacing

---

### 5. Type Hint Fixes (mypy - check only)

```bash
# Install mypy
pip install mypy

# Check for type issues (doesn't auto-fix)
mypy . --ignore-missing-imports

# Generate report
mypy . --ignore-missing-imports --html-report mypy-report
```

**Shows:**
- Missing type hints
- Type mismatches
- Invalid type usage

---

### 6. Python Docstring Formatting (docformatter)

```bash
# Install docformatter
pip install docformatter

# Format docstrings
docformatter --in-place --recursive .
```

**What docformatter Fixes:**
- Docstring quotes (""" vs ''')
- Docstring indentation
- Summary line formatting

---

### 7. Generate Fix Report

```markdown
# Lint Fix Report

## Files Modified

### Python Formatting (Black)
- Files formatted: [N]
- Lines changed: [N]

### Import Sorting (isort)
- Imports reorganized: [N] files
- Duplicate imports removed: [N]

### Unused Code Removal (autoflake)
- Unused imports removed: [N]
- Unused variables removed: [N]

### PEP 8 Fixes (autopep8)
- Whitespace fixes: [N]
- Indentation fixes: [N]

### Type Checking (mypy)
⚠️ Issues found: [N]
  - Missing type hints: [N] functions
  - Type mismatches: [N] locations

## Summary
✅ Auto-fixed: [N]+ issues
⚠️ Manual review needed: [N] type issues

## Next Steps
1. Review type hint warnings
2. Run tests to ensure nothing broken
3. Commit formatted code
```

---

### 8. Verify No Breaking Changes

```bash
# Run syntax check (for Python)
python -m py_compile **/*.py

# Run tests if available
pytest tests/ -v
# OR
python -m unittest discover
```

---

## All-in-One Script

Create a script `lint-all.ps1` (Windows) or `lint-all.sh` (Linux/Mac):

**Windows PowerShell:**
```powershell
Write-Host "🔧 Running linters and formatters..."

# Black
Write-Host "`n1. Formatting with Black..."
black . --line-length 100

# isort
Write-Host "`n2. Sorting imports..."
isort . --profile black

# autoflake
Write-Host "`n3. Removing unused code..."
autoflake --in-place --remove-all-unused-imports --remove-unused-variables --recursive .

# autopep8
Write-Host "`n4. Fixing PEP 8 issues..."
autopep8 --in-place --aggressive --recursive .

# mypy (check only)
Write-Host "`n5. Type checking..."
mypy . --ignore-missing-imports

Write-Host "`n✅ Linting complete!"
```

**Linux/Mac Bash:**
```bash
#!/bin/bash
echo "🔧 Running linters and formatters..."

echo "\n1. Formatting with Black..."
black . --line-length 100

echo "\n2. Sorting imports..."
isort . --profile black

echo "\n3. Removing unused code..."
autoflake --in-place --remove-all-unused-imports --remove-unused-variables --recursive .

echo "\n4. Fixing PEP 8 issues..."
autopep8 --in-place --aggressive --recursive .

echo "\n5. Type checking..."
mypy . --ignore-missing-imports

echo "\n✅ Linting complete!"
```

---

## Configuration Files

Create these in your project root for consistent behavior:

**pyproject.toml:**
```toml
[tool.black]
line-length = 100
target-version = ['py311']

[tool.isort]
profile = "black"
line_length = 100
```

**setup.cfg:**
```ini
[autoflake]
remove-all-unused-imports = true
remove-unused-variables = true

[mypy]
ignore_missing_imports = True
python_version = 3.11
```

---

## Pre-Commit Hook

Create `.git/hooks/pre-commit`:

```bash
#!/bin/sh
# Auto-format before commit

echo "Running auto-formatters..."
black . --line-length 100
isort . --profile black

# Add formatted files
git add -u

echo "✅ Code formatted"
```

Make executable:
```bash
chmod +x .git/hooks/pre-commit
```

---

## Quick One-Liner

```bash
# Windows PowerShell
black . && isort . --profile black && autoflake --in-place --remove-all-unused-imports --recursive .

# Linux/Mac Bash
black . && isort . --profile black && autoflake --in-place --remove-all-unused-imports --recursive .
```

**This workflow keeps Python code clean across ANY project!**

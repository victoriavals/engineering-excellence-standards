---
description: Check for outdated dependencies and suggest updates
---

# Dependency Check Workflow

**Purpose:** Monitor dependencies, check for updates, dan identify security vulnerabilities.

**Use Cases:**
- Monthly dependency review
- Security audit
- Before major releases
- After long development cycles

---

## Dependency Analysis

### 1. Python Dependencies

// turbo
```bash
# List all installed packages
pip list

# Check for outdated
pip list --outdated

# Export current
pip freeze > requirements_current.txt
```

**Report:**
```
📦 Python Packages

Installed: 25 packages
Outdated: 3 packages

Outdated Packages:
- fastapi: 0.100.0 → 0.109.0 (latest)
- httpx: 0.24.0 → 0.26.0 (latest)
- pydantic: 2.5.0 → 2.6.0 (latest)
```

---

### 2. Security Vulnerability Scan

```bash
# Install safety (if not installed)
pip install safety

#  Check for known vulnerabilities
safety check --json
```

**Report:**
```
🔒 Security Scan Results

Vulnerabilities Found: 1

CRITICAL:
- urllib3 (1.26.0)
  CVE-2021-33503
  Recommendation: Upgrade to >= 1.26.5
```

---

### 3. Node.js Dependencies (if applicable)

```bash
if (Test-Path "package.json") {
    # List outdated
    npm outdated
    
    # Security audit
    npm audit
}
```

---

### 4. Dependency Size Analysis

Check package sizes (important for Vercel):

```powershell
$packages = pip list --format=freeze | ForEach-Object {
    $pkg = ($_ -split '==')[0]
    $info = pip show $pkg 2>$null
    if ($info) {
        $size = ($info | Select-String "Size:").Line -replace "Size: ", ""
        [PSCustomObject]@{
            Package = $pkg
            Size = $size
        }
    }
} | Sort-Object {[int]($_.Size -replace '[^\d]', '')} -Descending

# Show top 10 largest
$packages | Select-Object -First 10
```

**Report:**
```
📊 Largest Packages (Top 10)

1. numpy: 45.2 MB
2. pandas: 38.1 MB
3. httpx: 2.1 MB
4. fastapi: 1.8 MB
5. pydantic: 1.5 MB
...

⚠️ Warning: numpy + pandas = 83MB (exceeds Vercel 50MB limit)
```

---

### 5. Unused Dependencies Detection

```powershell
# Get all imports from .py files
$imports = Select-String -Path "*.py" -Pattern "^(import|from)\s+(\w+)" | 
    ForEach-Object {
        if ($_.Line -match "^import\s+(\w+)") {
            $matches[1]
        } elseif ($_.Line -match "^from\s+(\w+)") {
            $matches[1]
        }
    } | Sort-Object -Unique

# Get installed packages
$installed = pip list --format=freeze | ForEach-Object {($_ -split '==')[0]}

# Find unused
$unused = $installed | Where-Object {$imports -notcontains $_}
```

**Report:**
```
📋 Potentially Unused Dependencies

- requests (installed but not imported)
- beautifulsoup4 (installed but not imported)

⚠️ Review before removing (may be used by other packages)
```

---

### 6. Update Recommendations

Ask user what to update:

```markdown
# Update Recommendations

## Critical (Security)
- [ ] urllib3: 1.26.0 → 1.26.5 (CVE fix)

## High Priority  
- [ ] fastapi: 0.100.0 → 0.109.0 (bug fixes, features)
- [ ] pydantic: 2.5.0 → 2.6.0 (performance)

## Low Priority
- [ ] httpx: 0.24.0 → 0.26.0 (minor improvements)

## Optional
- [ ] Remove unused: requests, beautifulsoup4

---

**Update Commands:**
```bash
# Critical
pip install --upgrade urllib3

# High priority
pip install --upgrade fastapi pydantic

# All at once
pip install --upgrade -r requirements.txt

# Remove unused
pip uninstall requests beautifulsoup4
```

---

### 7. Test After Updates

After updating:

```bash
// turbo
# Update requirements.txt
pip freeze > requirements.txt

# Run tests
pytest tests/

# Start server and test
python app.py
```

---

### 8. Generate Dependency Report

```markdown
# Dependency Health Report

## Summary
- Total packages: 25
- Outdated: 3 
- Security vulnerabilities: 1 (critical)
- Unused packages: 2
- Total size: 125 MB

## Action Items
1. ❗ Update urllib3 (security vulnerability)
2. Consider updating fastapi and pydantic
3. Remove unused packages to reduce size
4. Review pandas/numpy if not essential (size concern)

## Vercel Deployment
Current size: 125 MB
⚠️ Exceeds 50MB limit

**Recommendations:**
- Remove numpy + pandas if not needed
- Consider lightweight alternatives
- Use Vercel's larger function size (Pro plan)

## Next Review
Scheduled: [30 days from now]
```

**Run `/dependency-check` monthly to keep dependencies healthy!**

---
description: Remove all temporary test files and caches (Universal)
---

# Clean Temporary Files Workflow (Universal)

**Purpose:** Clean up workspace dengan remove temporary files, caches, dan test artifacts. Works for ANY project (Python, Node.js, etc).

**Use Cases:**
- After testing session
- Before committing clean code
- Reduce repo size
- Fresh start debugging

---

## Cleanup Steps

### 1. List Temporary Files First

Show user what will be removed:

**Python Caches:**
```bash
Get-ChildItem -Recurse -Directory -Filter "__pycache__"
Get-ChildItem -Recurse -Filter "*.pyc"
Get-ChildItem -Recurse -Filter "*.pyo"
```

**Test Caches:**
```bash
Get-ChildItem -Recurse -Directory -Filter ".pytest_cache"
Get-ChildItem -Directory -Filter ".coverage"
```

**Node.js (if applicable):**
```bash
Get-ChildItem -Directory -Filter "node_modules" | Measure-Object | Select-Object Count
```

**Build Artifacts:**
```bash
Get-ChildItem -Recurse -Directory -Filter "build"
Get-ChildItem -Recurse -Directory -Filter "dist"
```

**Ask User:** "Files ini akan dihapus. Lanjut?"

---

### 2. Remove Python Caches

// turbo
```bash
# Remove __pycache__ directories
Get-ChildItem -Recurse -Directory -Filter "__pycache__" | Remove-Item -Recurse -Force

# Remove .pyc files
Get-ChildItem -Recurse -Filter "*.pyc" | Remove-Item -Force

# Remove .pyo files
Get-ChildItem -Recurse -Filter "*.pyo" | Remove-Item -Force
```

---

### 3. Remove Test Caches

// turbo
```bash
# pytest cache
Get-ChildItem -Recurse -Directory -Filter ".pytest_cache" | Remove-Item -Recurse -Force

# coverage files
Get-ChildItem -Filter ".coverage" | Remove-Item -Force
Get-ChildItem -Recurse -Directory -Filter "htmlcov" | Remove-Item -Recurse -Force

# unittest cache  
Get-ChildItem -Recurse -Directory -Filter ".tox" | Remove-Item -Recurse -Force
```

---

### 4. Remove Build Artifacts

```bash
# Python build
Get-ChildItem -Recurse -Directory -Filter "build" | Remove-Item -Recurse -Force
Get-ChildItem -Recurse -Directory -Filter "dist" | Remove-Item -Recurse -Force
Get-ChildItem -Recurse -Directory -Filter "*.egg-info" | Remove-Item -Recurse -Force

# Compiled files
Get-ChildItem -Recurse -Filter "*.so" | Remove-Item -Force
Get-ChildItem -Recurse -Filter "*.dylib" | Remove-Item -Force
Get-ChildItem -Recurse -Filter "*.dll" | Remove-Item -Force
```

---

### 5. Frontend Build Artifacts (if applicable)

```bash
# Next.js
Get-ChildItem -Directory -Filter ".next" | Remove-Item -Recurse -Force

# Vite/React build
Get-ChildItem -Directory -Filter "dist" | Remove-Item -Recurse -Force

# Gatsby
Get-ChildItem -Directory -Filter ".cache" | Remove-Item -Recurse -Force
Get-ChildItem -Directory -Filter "public" | Remove-Item -Recurse -Force
```

---

### 6. Remove Log Files (Optional)

Ask: "Mau hapus log files juga?"

```bash
# Remove .log files
Get-ChildItem -Recurse -Filter "*.log" | Remove-Item -Force

# Keep important logs, only remove debug/temp
Get-ChildItem -Recurse -Filter "debug*.log" | Remove-Item -Force
Get-ChildItem -Recurse -Filter "temp*.log" | Remove-Item -Force
Get-ChildItem -Recurse -Filter "test*.log" | Remove-Item -Force
```

---

### 7. IDE/Editor Caches

```bash
# VSCode
Get-ChildItem -Directory -Filter ".vscode" | Where-Object {Test-Path "$_\.history"} | Remove-Item -Recurse -Force

# JetBrains IDEs
Get-ChildItem -Recurse -Directory -Filter ".idea" | Remove-Item -Recurse -Force

# Vim swap files
Get-ChildItem -Recurse -Filter "*.swp" | Remove-Item -Force
Get-ChildItem -Recurse -Filter "*.swo" | Remove-Item -Force
```

---

### 8. OS-Specific Files

```bash
# macOS
Get-ChildItem -Recurse -Filter ".DS_Store" | Remove-Item -Force

# Windows
Get-ChildItem -Recurse -Filter "Thumbs.db" | Remove-Item -Force
Get-ChildItem -Recurse -Filter "desktop.ini" | Remove-Item -Force
```

---

### 9. Git Cleanup (Optional)

Ask: "Mau cleanup git cache juga?"

```bash
# Remove untracked files (dry-run first)
git clean -n

# Ask confirmation, then:
git clean -fd

# Remove ignored files too
git clean -fdx
```

---

### 10. Generate Cleanup Report

```markdown
# Cleanup Report

## Removed Items

### Python Caches
- __pycache__ directories: [N] removed
- .pyc/.pyo files: [N] removed
- Space freed: [XX]MB

### Test Caches
- .pytest_cache: [N] removed
- Coverage files: [N] removed

### Build Artifacts
- build/dist directories: [N] removed
- *.egg-info: [N] removed

### Frontend Artifacts (if applicable)
- .next/dist: [N] removed
- node_modules: (kept)

### Log Files
- *.log files: [N] removed

### IDE Caches
- .idea directories: [N] removed
- swap files: [N] removed

### OS Files
- .DS_Store: [N] removed
- Thumbs.db: [N] removed

## Total Space Freed
[XXX]MB

## Status
✅ Workspace cleaned successfully
```

---

## Quick Commands for Different Languages

### Python Only
```powershell
Get-ChildItem -Recurse -Directory -Filter "__pycache__" | Remove-Item -Recurse -Force
Get-ChildItem -Recurse -Filter "*.pyc" | Remove-Item -Force
Get-ChildItem -Recurse -Directory -Filter ".pytest_cache" | Remove-Item -Recurse -Force
```

### Node.js/JavaScript
```bash
# Remove node_modules (LARGE - ask first!)
Remove-Item -Recurse -Force node_modules
npm install  # Reinstall if needed
```

### All Languages (Full Clean)
```bash
# Run all cleanup steps
```

---

## Safety Features

1. **Dry Run First:** Always show what will be deleted
2. **User Confirmation:** Ask before deleting important items
3. **Selective Cleanup:** User can choose categories
4. **Protected Files:** Never delete:
   - `.env` files
   - Source code
   - Production logs
   - `.git` directory

---

## Platform-Specific Commands

**Windows PowerShell:**
```powershell
Get-ChildItem -Recurse -Directory -Filter "__pycache__" | Remove-Item -Recurse -Force
```

**Linux/Mac Bash:**
```bash
find . -type d -name "__pycache__" -exec rm -rf {} +
find . -name "*.pyc" -delete
```

**Cross-Platform (Python):**
```python
import os
import shutil

for root, dirs, files in os.walk('.'):
    if '__pycache__' in dirs:
        shutil.rmtree(os.path.join(root, '__pycache__'))
```

**This workflow works for ANY project type!**

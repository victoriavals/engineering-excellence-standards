---
description: Interactive Git workflow - add, commit, push per file atau batch
---

# Git Command Workflow (Universal)

**Purpose:** Streamline git operations dengan interactive prompts untuk add, commit, dan push files. Works for ANY project.

**Use Cases:**
- Commit file satu per satu dengan message yang spesifik
- Batch commit multiple files dengan satu message
- Push changes ke remote repository
- Git workflow lengkap dari add sampai push

---

## Workflow Steps

### 1. Check Git Status
Tampilkan current git status untuk lihat modified/untracked files

```bash
git status
```

### 2. Ask User Preference
Tanyakan kepada user:
- "Mau commit per file atau batch semua file?"
- "File mana yang mau di-commit?" (kalau per file)

### 3. Add Files

**Option A: Per File**
```bash
git add <filename>
```
Lakukan per file yang user pilih

**Option B: Batch All**
```bash
git add .
```
Add semua modified files sekaligus

### 4. Commit with Message

**Per File Commit:**
Untuk setiap file yang di-add, tanyakan commit message:
```bash
git commit -m "[user's message for this file]"
```

**Batch Commit:**
Tanyakan satu commit message untuk semua files:
```bash
git commit -m "[user's single message for all files]"
```

### 5. Push to Remote
Tanyakan: "Mau push ke remote sekarang?"

If yes:
```bash
git push
```

If user mau push ke branch tertentu:
```bash
git push origin <branch-name>
```

### 6. Final Confirmation
Report hasil:
- Files committed: [list]
- Commit messages: [list]
- Push status: Success/Not pushed yet

---

## Example Commands Output Format

**Command akan di-generate dalam satu bash block, biar user dapat copy-paste langsung:**

```bash
# Example output for per-file workflow
git add file1.py
git commit -m "feat: add user authentication"
git add file2.py  
git commit -m "fix: resolve login bug"
git push
```

```bash
# Example output for batch workflow
git add .
git commit -m "feat: add multiple endpoints and tests"
git push
```

---

## Conventional Commits Format

Suggest proper commit message format:

- `feat: description` — New feature
- `fix: description` — Bug fix
- `docs: description` — Documentation only
- `style: description` — Code style (formatting, semicolons, etc)
- `refactor: description` — Code refactoring
- `perf: description` — Performance improvement
- `test: description` — Adding tests
- `chore: description` — Maintenance tasks
- `ci: description` — CI/CD changes

**Examples:**
```
feat: add user authentication endpoint
fix: resolve memory leak in cache service
docs: update API documentation
refactor: extract validation logic to utility
test: add unit tests for user service
```

---

## Tips

1. **Per File** cocok untuk:
   - Changes yang tidak related
   - Precise commit history
   - Easy rollback per feature

2. **Batch** cocok untuk:
   - Related changes
   - Quick commits
   - Cleanup commits

3. **Check Before Commit:**
   - Run tests if available
   - Review diffs: `git diff`
   - Check branch: `git branch`

**This workflow works for ANY git repository!**

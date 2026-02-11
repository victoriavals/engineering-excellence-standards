---
description: Daily morning health check for project
---

# Daily Health Check Workflow

**Purpose:** Morning routine check untuk assess overall project health, dependencies, dan potential issues.

**Use Cases:**
- Daily standup preparation
- Weekly sprint check
- After vacation/weekend
- Project health monitoring

---

//

 turbo-all

## Health Check Areas

### 1. Git Status Check

```bash
# Uncommitted changes?
git status --short

# Unpushed commits?
git log origin/main..HEAD --oneline
```

**Report:**
- Uncommitted files: [N]
- Untracked files: [N]
- Commits ahead of remote: [N]

---

### 2. Dependency Status

```bash
# Python
pip list --outdated

# Node.js (if exists)
npm outdated
```

**Report:**
- Outdated packages: [N]
- Critical updates: [list]

---

### 3. Database Connection

Run `/db-status` workflow check

**Report:**
- ✅ Supabase connection: OK
- Total records: [N]
- Recent activity: [timestamp]

---

### 4. API Health

Run `/api-health` workflow check

**Report:**
- ✅ All endpoints responding
- Average response time: [XX]ms

---

### 5. Security Quick Scan

```bash
# Check for new hardcoded secrets
Select-String -Path "*.py" -Pattern '(api_key|password|token)\s*=\s*["\'][^"\'get]+["\']' -CaseSensitive

# Check .env not committed
git ls-files | Select-String "\.env$"
```

**Report:**
- New secrets found: [N]
- .env in git: No ✅

---

###6. Test Suite Status

```bash
# Run quick tests
pytest tests/ --collect-only  # Just count, don't run

# Get last test run results (if available)
```

**Report:**
- Total tests: [N]
- Last run: [timestamp]
- Status: Pass/Fail

---

### 7. Code Quality Metrics

```powershell
# Count modified files lately
$recentFiles = git diff --name-only HEAD~10..HEAD

# Lines changed
$additions = (git diff --stat HEAD~10..HEAD | Select-String "insertion")
```

**Report:**
- Files changed (last 10 commits): [N]
- Lines added: [N]
- Lines removed: [N]

---

### 8. Generate Daily Health Score

Calculate score based on checks:

```
Health Calculation:
- Git clean: 15 points
- Dependencies updated: 20 points
- Database healthy: 20 points
- API responsive: 20 points
- No security issues: 15 points
- Tests passing: 10 points

Total: /100 points
```

**Report:**
```markdown
# Daily Health Report - [Date]

## 📊 Health Score: [XX]/100

### Git Status: 🟢 Clean
- No uncommitted changes
- No unpushed commits
- Branch: up-to-date with remote

### Dependencies: 🟡 Updates Available
- Outdated: 3 packages
- Critical: None
- Recommendation: Review and update

### Database: 🟢 Healthy
- Connection: ✅ OK
- Records: 1,234
- Recent activity: 2 min ago

### API: 🟢 Operational
- All endpoints: ✅ responding
- Average latency: 45ms
- Status: Excellent

### Security: 🟢 Secure
- No hardcoded secrets
- .env properly ignored
- Dependencies: No known vulnerabilities

### Tests: 🟢 Passing
- Total: 42 tests
- Last run: 2 hours ago
- Status: All passing

## 📋 Action Items
1. Update 3 outdated dependencies (low priority)
2. Consider adding tests for new endpoint

## Overall Status
🟢 HEALTHY - No critical issues

Good to start the day! 🚀
```

---

## Workflow Variants

**Quick Check (2 min):**
- Steps 1, 3, 4 only

**Full Audit (10 min):**
- All 8 steps

**Weekly Deep Dive:**
- Add `/security-scan` and `/code-review`

---

## Automation Tip

Schedule this workflow to run automatically:

```powershell
# Windows Task Scheduler
# Run every morning at 9 AM
# Command: powershell.exe -Command "cd C:\your\project; antigravity run /daily-health"
```

**This workflow keeps you informed about project health daily!**

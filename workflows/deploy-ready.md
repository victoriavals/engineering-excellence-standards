---
description: Verify app is ready for production deployment
---

# Deploy Ready Workflow

**Purpose:** Comprehensive pre-deployment check untuk Vercel atau platform lain, ensuring app meets all requirements.

**Use Cases:**
- Before `vercel --prod`
- Pre-deployment validation
- Production readiness audit
- Size and configuration verification

---

## Pre-Deployment Checklist

### 1. Run Quick Test First

```
Execute /quick-test workflow
```

**Prerequisite:** All tests must pass before continuing.

---

### 2. Check Package Sizes (Vercel Limit: ~50MB)

// turbo
**Python Dependencies:**
```bash
# Get size of all packages
pip list --format=freeze | ForEach-Object {
    $pkg = ($_ -split '==')[0]
    pip show $pkg | Select-String "Location|Size"
}

# Estimate total deployment size
```

**Check for Heavy Packages:**
- LangChain (>100MB) ❌
- TensorFlow/PyTorch (>500MB) ❌
- pandas with deps (~100MB) ⚠️

**Report:**
```
Total estimated size: [XX]MB
Status: ✅ Under 50MB limit / ⚠️ Close to limit / ❌ Over limit
```

---

### 3. Verify vercel.json Configuration

```bash
# Check if vercel.json exists
if (Test-Path "vercel.json") {
    Get-Content vercel.json | ConvertFrom-Json
} else {
    Write-Host "⚠️ vercel.json not found!"
}
```

**Verify Settings:**
- `buildCommand` specified
- `outputDirectory` correct
- `framework` detected
- Routes/rewrites configured (if needed)

---

### 4. Environment Variables Audit

**Check .env.example:**
```bash
Get-Content .env.example
```

**Verify Documentation:**
- All required vars listed
- Descriptions provided
- No default secrets

**Remind User:**
```
⚠️ Remember to set these in Vercel Dashboard:
1. Go to Project Settings → Environment Variables
2. Add each variable from .env.example
3. Set for: Production, Preview, Development
```

---

### 5. Security Check

// turbo
**Scan for Hardcoded Secrets:**
```bash
# Search for potential secrets in code
Select-String -Path "*.py" -Pattern '(api_key|password|secret|token)\s*=\s*["\'](?!.*getenv)' -CaseSensitive
```

**Check .gitignore:**
```bash
# Verify .env is ignored
Select-String -Path ".gitignore" -Pattern "\.env$"
```

**Report:**
```
✅ No hardcoded secrets found
✅ .env properly ignored
✅ All secrets use environment variables
```

---

### 6. Dependency Lock File

**Python:**
```bash
# Ensure requirements.txt is updated
pip freeze > requirements.txt
```

**Node.js (if exists):**
```bash
# Ensure package-lock.json exists
if (Test-Path "package.json") {
    if (-not (Test-Path "package-lock.json")) {
        Write-Host "⚠️ Run: npm install"
    }
}
```

---

### 7. README & Documentation Check

Verify README has:
- Deployment instructions
- Environment variables section
- API endpoints documented
- Setup guide

**Quick Scan:**
```bash
Select-String -Path "README.md" -Pattern "(deploy|environment|api endpoints)" -CaseSensitive
```

---

### 8. Final Confirmation

Present deployment readiness report:

```markdown
# Deployment Readiness Report

## ✅ Tests
- All syntax checks passed
- All unit tests passed  
- Server starts successfully

## ✅ Build Requirements  
- Package size: [XX]MB (under 50MB limit)
- vercel.json configured
- Dependencies locked

## ✅ Security
- No hardcoded secrets
- .env ignored in git
- All vars use environment variables

## ✅ Configuration
- Environment variables documented
- README up to date

## 📋 Action Items
1. Set environment variables in Vercel Dashboard
2. Review last commits
3. Ensure correct branch selected

---

Ready to deploy? Run:
```bash
vercel --prod
```

Or ask: "Mau saya deploy sekarang?"
```

---

### 9. Deploy (Optional)

If user confirms:

// turbo
```bash
vercel --prod
```

**Capture Output:**
- Deployment URL
- Build time
- Any errors/warnings

---

### 10. Post-Deployment Verification

After deployment:

```bash
# Test production endpoint
$prodUrl = "https://your-app.vercel.app"
curl "$prodUrl/api/health"
```

**Report:**
```
✅ Deployment successful
✅ Production URL: [link]
✅ Health check passed

Next steps:
1. Test main features in production
2. Monitor logs in Vercel dashboard
3. Check analytics/errors
```

---

## Summary

This workflow ensures:
- ✅ Code quality (via /quick-test)
- ✅ Size limits respected
- ✅ Configuration correct
- ✅ Security best practices
- ✅ Documentation complete
- ✅ Smooth deployment process

**Run `/deploy-ready` before every production deploy to avoid failures!**

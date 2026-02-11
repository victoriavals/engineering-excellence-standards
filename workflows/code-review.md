---
description: AI-powered code review of recent changes (Universal)
---

# Code Review Workflow (Universal)

**Purpose:** AI reviews your recent code changes against best practices and coding standards. Works for ANY language if you have coding rules defined.

**Use Cases:**
- Pre-commit self-review
- Before creating PR
- After implementing feature
- Code quality improvement

---

## Review Process

### 1. Get Recent Changes

Ask user: "Mau review apa?"
- Last commit
- Last N commits
- Uncommitted changes
- Specific files

```bash
# Last N commits (default 1)
git log -1 --name-only --pretty=format:""

# Uncommitted changes
git diff --name-only

# Specific commit
git show <commit-hash> --name-only

# Files in last N commits
git log -n 5 --name-only --pretty=format:""
```

---

### 2. For Each Modified File

Read file content and analyze against:

**Universal Code Quality:**
- ✅ Consistent formatting?
- ✅ Clear variable/function names?
- ✅ Appropriate comments/documentation?
- ✅ Error handling present?
- ✅ No code duplication?
- ✅ Following language conventions?

**Language-Specific (detect from extension):**

**Python (.py):**
- Type hints present?
- Docstrings (Google/NumPy style)?
- PEP 8 compliance?
- Using proper logging (not print)?

**JavaScript/TypeScript (.js, .ts):**
- JSDoc comments?
- TypeScript types defined?
- ESLint compliant?
- Async/await used properly?

**Go (.go):**
- Error handling with if err != nil?
- Proper package comments?
- gofmt compliance?

**PHP (.php):**
- PHPDoc blocks?
- PSR standards?
- Type declarations?

---

### 3. Security Analysis

**Universal Security Checks:**
- ❌ Hardcoded secrets? (API keys, passwords, tokens)
- ❌ SQL injection risks?
- ❌ Missing input validation?
- ❌ Unsafe file operations?
- ❌ Weak crypto usage?

**Patterns to Flag:**
```
# Any language
password = "hardcoded123"
api_key = "sk-1234567890"
token = "Bearer xyz..."

# SQL injection patterns
query = f"SELECT * FROM users WHERE id = {user_id}"
db.execute("SELECT * WHERE name = '" + name + "'")

# Command injection
os.system(f"rm {filename}")
exec(user_input)
```

---

### 4. Performance Analysis

**Common Issues:**
- ⚠️ N+1 query problems?
- ⚠️ Blocking I/O in async context?
- ⚠️ Inefficient loops?
- ⚠️ Missing database indexes hints?
- ⚠️ Large data loading without pagination?

---

### 5. Testing Coverage

**Check:**
- ❓ Tests added for new features?
- ❓ Edge cases covered?
- ❓ Error scenarios tested?
- ❓ Integration tests present?

---

### 6. Generate Review Report

```markdown
## File: [filename]

### Issues Found

#### ❌ Critical (Must Fix)
1. **Line 45:** Hardcoded API key
   ```[language]
   API_KEY = "sk-1234567890"  # ❌ Bad
   ```
   **Fix:** Use environment variables
   ```[language]
   API_KEY = os.getenv("API_KEY")
   ```

2. **Line 67:** No error handling
   ```[language]
   data = fetchData(id)  # What if fails?
   return data
   ```
   **Fix:** Add try/catch or error checking

#### ⚠️ Warnings (Should Fix)
1. **Line 23:** Missing documentation
   ```[language]
   function processData(data) {  # No description
   ```
   **Fix:** Add JSDoc/docstring

2. **Line 89:** Console.log in production code
   ```[language]
   console.log("Debug:", value)  # ⚠️
   ```
   **Fix:** Use proper logging library

#### ✅ Good Practices Found
- Clear function names
- Proper code structure
- Input validation present
- Async/await used correctly

### Recommendations
1. Add documentation to all public functions
2. Extract repeated logic to utility function
3. Consider adding rate limiting
4. Add integration tests

### Overall Score: 7/10
**Rating:** Good, but needs security and error handling improvements
```

---

### 7. Detect Anti-Patterns

**Universal Anti-Patterns:**

```
# Magic numbers
if (value > 100) { ... }  # What is 100?
# Better: if (value > MAX_RETRIES) { ... }

# Deep nesting
if (a) {
  if (b) {
    if (c) {
      if (d) {  # Too deep!
```

**Language-Specific:**

**Python:**
- Bare `except:` clauses
- Mutable default arguments
- Missing `await` in async functions

**JavaScript:**
- Using `var` instead of `const`/`let`
- Not handling Promise rejections
- Mutating function parameters

**Go:**
- Ignoring errors: `_, err := ...`
- Not closing resources (defer)

---

### 8. Suggest Improvements

For each file, provide concrete before/after examples tailored to the language.

---

### 9. Generate Summary

```markdown
# Code Review Report

## Files Reviewed: [N]

### Summary
- Lines changed: +[N], -[N]
- Issues found: [N] ([N] critical, [N] warnings)
- Security issues: [N]
- Performance concerns: [N]

## Critical Issues ❌
[List with file:line references]

## Warnings ⚠️
[List with file:line references]

## Good Practices Found ✅
- [List positive findings]

## Recommendations
1. **Immediate:** Fix [N] critical security issues
2. **Before commit:** Add documentation/tests
3. **Nice to have:** Refactor for better readability
4. **Future:** Consider [architectural suggestions]

## Overall Code Quality
📊 Score: [N]/10

**Status:** ⚠️ Needs improvements / ✅ Ready to merge

**Estimated Fix Time:** [time estimate]
```

---

### 10. Interactive Fixes

Ask: "Mau saya bantu fix issues yang ditemukan?"

If yes:
1. Show the problem
2. Show the fix
3. Ask to apply
4. Apply and verify syntax

---

## Customization

**If GlobalRule.md exists in project:**
- Review against those specific standards
- Check for project-specific patterns

**If no coding standards:**
- Use universal best practices
- Language-specific conventions
- Industry standards (PEP 8, ESLint, PSR, etc.)

---

## Review Checklist

Universal checklist:

```markdown
- [ ] All functions documented
- [ ] No hardcoded secrets
- [ ] Error handling present
- [ ] Input validation added
- [ ] No security vulnerabilities
- [ ] Performance optimized
- [ ] Tests added for new features
- [ ] Code follows style guide
- [ ] No code duplication
- [ ] Clear naming conventions
```

**Use this workflow before every commit to maintain high code quality across ANY project!**

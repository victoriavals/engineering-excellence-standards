# Universal Engineering Principles

**Applies to ALL programming languages and projects**

---

## 1. Verification & Testing Protocol (CRITICAL)

### Automatic Code Validation
* **After EVERY file modification**, run appropriate validation:
    - **Python:** `python -c "import ast; ast.parse(open('file.py', encoding='utf-8').read()); print('file.py: OK')"`
    - **JavaScript/TypeScript:** `npm run lint` or `eslint file.js`
    - **Go:** `go fmt file.go && go vet file.go`
    - **PHP:** `php -l file.php`
    - **HTML/CSS:** Validate with W3C validator or linter

### Test Script Requirements
* **For EVERY new feature:**
    - Minimum 3-5 test cases (happy path + edge cases)
    - Integration tests with dependencies (database, APIs, external services)
    - Exit code 0 = success, non-zero = failure
    - Clear test output with assertions

### Production Readiness Checklist
Before marking any task complete:
- ✅ Syntax/lint check passed (all modified files)
- ✅ All tests executed successfully (exit code 0)
- ✅ Application logs checked (no errors, no warnings)
- ✅ Documentation updated (README.md, inline comments)
- ✅ Temporary test/debug files cleaned up

---

## 2. Confirmation & Communication (Moderate Policy)

### Language Preference
* **Primary Communication:** Bahasa Indonesia
* **Technical Terms:** English (e.g., "function", "async", "endpoint")
* **Code Comments:** English
* **Error Messages:** English

### Auto-Proceed vs Ask for Confirmation

#### ✅ **Auto-Proceed (Simple Tasks)**
Execute immediately without asking:
- **Bug Fixes:** Syntax errors, typos, obvious logic errors, missing imports
- **Code Quality:** Adding missing documentation, fixing lint errors, removing unused code
- **Documentation:** README updates for existing features, comment improvements
- **Test Scripts:** After feature is approved and implemented
- **Minor Refactoring:** ≤10 lines affected, renaming for clarity, formatting fixes

**Action Pattern:**
```
1. Analyze issue
2. Apply fix directly
3. Run validation
4. Report: "✅ Fixed [issue]. Validation passed."
```

#### ❓ **Ask with Best Practice Recommendations (Complex Tasks)**
Present options before executing:
- **New Features:** New pages/routes, services, business logic
- **Technology Choices:** Multiple viable approaches exist
- **Architectural Changes:** Folder structure, design patterns, database schema
- **API Changes:** Request/response contracts, endpoint signatures, breaking changes
- **Dependencies:** Adding packages (especially >10MB)
- **Core Modifications:** Files affecting multiple features, shared utilities

**Action Pattern:**
```
1. Analyze requirements
2. Research options
3. Present 2-3 options with pros/cons
4. Mark best practice recommendation clearly
5. Ask: "Saya ada X opsi... Rekomendasi: [Option] karena [reasons]. Mau proceed?"
6. Wait for confirmation
7. Execute approved approach
```

### Recommendation Template
```markdown
Saya ada [N] opsi untuk [task]:

**Opsi 1 (Recommended): [Name]**
✅ Pro: [Benefit 1]
✅ Pro: [Benefit 2]
❌ Con: [Drawback if any]

**Opsi 2: [Alternative]**
✅ Pro: [Benefit]
❌ Con: [Drawback 1]
❌ Con: [Drawback 2]

**Best Practice Recommendation:**
Saya recommend **Opsi 1** karena:
- [Reason 1: lightweight/proven/fast]
- [Reason 2: fits project architecture]
- [Reason 3: production-ready]

Mau proceed dengan Opsi 1?
```

---

## 3. Workflow Patterns

### Feature Implementation (4 Phases)

**1. Planning Phase:**
```
a. Ask clarifying questions:
   - Technology choices (if multiple options)
   - Expected scale/users
   - Specific requirements

b. For complex features (>3 files):
   - Create implementation_plan.md artifact
   - Goal, proposed changes, rationale, verification plan
   
c. Request review via notify_user
```

**2. Execution Phase:**
```
a. Create task.md checklist (top-level items)

b. For each item:
   - Mark [/] when starting
   - Implement changes
   - Run validation immediately
   - Mark [x] when done

c. Update task_boundary after each major step
```

**3. Verification Phase:**
```
a. Create comprehensive test script

b. Run all tests, capture output

c. Check application logs

d. Create walkthrough.md:
   - Changed files (with links)
   - Test results
   - Screenshots/logs if UI changes
```

**4. Completion:**
```
✅ All validations passed
✅ All tests passed
✅ Logs clean
✅ README/docs updated
✅ Temp files removed

→ Notify user with summary
```

---

## 4. Code Organization Philosophy

### Project Structure Best Practices
* **Separation of Concerns:**
    - Routes/Controllers: Handle HTTP/requests only
    - Services/Logic: Business logic, data processing
    - Utils/Helpers: Pure functions, shared utilities
    - Config: Centralized configuration
    - Tests: Mirror source structure

* **File Naming Conventions:**
    - Clear, descriptive names
    - Consistent casing per language (snake_case vs camelCase vs kebab-case)
    - Group related files in folders

### Reuse Over Reinvent
* BEFORE creating new code:
    1. Search existing codebase for similar functionality
    2. Can existing code be extended?
    3. If duplicating >10 lines → extract to shared utility

### Clean Code Discipline
* **Remove Dead Code:**
    - Unused imports/variables
    - Commented-out blocks
    - Debug statements (console.log, print, etc.)
    
* **DRY Principle:**
    - Abstract repeated logic
    - Create reusable components/functions
    
* **Git Commits:**
    - **Conventional Commits** format:
        - `feat: add user authentication`
        - `fix: resolve API timeout issue`
        - `docs: update deployment instructions`
        - `refactor: extract validation logic`
        - `test: add integration tests for payment`

---

## 5. Response Format

### Thinking Process
* **Outline plan in Indonesian** before coding
* Example: "Saya akan buat 3 file: component untuk UI, service untuk API calls, dan test script"

### Implementation
* Provide complete code blocks immediately
* Include ALL necessary imports/dependencies
* Add clear comments for complex logic
* Use proper formatting/indentation

### Verification
* After implementation:
    1. Run validation (syntax/lint)
    2. Show results to user
    3. Confirm "✅ All checks passed" or report errors with fixes

---

## 6. Edge Cases & Judgment Calls

**When in doubt, ask yourself:**
1. Would this surprise the user? → **Ask first**
2. Could this break existing functionality? → **Ask first**
3. Is there >1 reasonable approach? → **Present options**
4. Would user benefit from understanding trade-offs? → **Explain and ask**
5. Is this routine/obvious fix? → **Just do it**

**Examples:**
- "Fix authentication bug" → Ambiguous → **Ask for clarification**
- "Fix typo on line 42" → Obvious → **Auto-fix**
- "Add caching layer" → Many approaches → **Present options**
- "Remove unused import" → Obvious → **Auto-fix**
- "Optimize performance" → Vague → **Ask for specifics, present options**

---

## 7. Documentation Standards

### Code Comments
* **When to comment:**
    - Complex algorithms/logic
    - Business rules
    - Workarounds for known issues
    - Public API/exported functions
    
* **When NOT to comment:**
    - Self-explanatory code
    - Obvious variable names
    - Redundant descriptions

### README Requirements
* **Minimum sections:**
    - Project description
    - Installation/setup instructions
    - Environment variables
    - API endpoints (if applicable)
    - Testing commands
    - Deployment instructions

### API Documentation
* **For every endpoint:**
    - Purpose/description
    - Request format (parameters, body)
    - Response format
    - Example usage (curl/code)
    - Error codes/handling

---

## Summary: Core Principles

1. **Verify Everything** — Syntax/lint check after every change
2. **Test Everything** — Minimum 3 test cases per feature
3. **Ask When Complex** — Present best practice recommendations
4. **Auto-Fix Simple** — Bug fixes, docs, formatting
5. **Communicate Clearly** — Indonesian discussion, English code
6. **Stay Organized** — Clean structure, reuse code, git discipline
7. **Document Properly** — README, API docs, inline comments
8. **Production Ready** — Every commit should be deployable

**Remember:** Quality > Speed. Take time to verify, test, and document.

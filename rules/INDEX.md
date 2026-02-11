# Engineering Rules Index

**Multi-language development rules organized by scope**

---

## 📚 Available Rule Files

### 1. Universal Principles (All Languages)
**File:** `universal-principles.md`

**Applies to:** Python, JavaScript, TypeScript, Go, PHP, HTML, CSS

**Covers:**
- Verification & testing protocol
- Confirmation & communication policy
- Workflow patterns (Planning → Execution → Verification)
- Code organization philosophy
- Response format
- Documentation standards

---

### 2. Frontend Web Development
**File:** `frontend-web.md`

**Applies to:** HTML, CSS, JavaScript, TypeScript, React, Vue, Next.js, Vite

**Priority Focus:** Deployment

**Covers:**
- TypeScript strict mode
- Component best practices (React/Vue)
- CSS/styling standards (BEM, CSS Modules, Tailwind)
- Testing (Jest/Vitest)
- **Deployment (Vercel/Netlify)** — Priority focus
- Performance optimization
- Security & accessibility
- Error handling & logging

---

### 3. Backend API Design
**File:** `backend-api-design.md`

**Applies to:** Python, Go, PHP (any backend language)

**Priority Focus:** API Design Patterns

**Covers:**
- **REST API design principles** — Priority focus
- HTTP status codes & URL structure
- Request/response format standards
- Authentication & authorization (JWT, API keys)
- Input validation & sanitization
- Error handling patterns
- Database best practices
- Pagination & filtering
- Rate limiting
- API documentation (OpenAPI/Swagger)

---

### 4. Python Backend Specific
**File:** `python-backend.md`

**Applies to:** Python (FastAPI, Flask, Django)

**Covers:**
- Core architecture & file structure
- Strict typing (non-negotiable type hints)
- Documentation (Google-style docstrings)
- Logging strategy (no print statements)
- Error handling & resilience
- PEP 8+ conventions
- AI & LangChain policy
- Async/await patterns
- FastAPI best practices
- Testing with pytest + httpx

---

## 🎯 How to Use These Rules

### When Coding Python Backend:
1. Read `universal-principles.md` (foundation)
2. Read `backend-api-design.md` (API patterns)
3. Read `python-backend.md` (Python-specific)

### When Coding Frontend (React/Vue/TS):
1. Read `universal-principles.md` (foundation)
2. Read `frontend-web.md` (frontend-specific)

### When Coding Go/PHP Backend:
1. Read `universal-principles.md` (foundation)
2. Read `backend-api-design.md` (API patterns)
3. Apply language-specific conventions from official style guides

---

## 📝 Quick Reference

### All Projects MUST:
✅ Run syntax/lint checks after every change  
✅ Write minimum 3-5 test cases per feature  
✅ Create implementation plans for complex features (>3 files)  
✅ Use conventional commits (feat/fix/docs/refactor/test)  
✅ Clean up temporary files after completion  
✅ Update README/documentation  

### AI MUST:
✅ Auto-proceed for simple tasks (bug fixes, docs, formatting)  
✅ Ask with best practice recommendations for complex tasks  
✅ Present 2-3 options with pros/cons when multiple approaches exist  
✅ Communicate in Bahasa Indonesia (technical terms in English)  
✅ Verify every change (syntax check + tests)  

---

## 🔄 Update History

- **2026-02-11:** Created multi-language rule structure
    - Split from monolithic GlobalRule.md
    - Added frontend-web.md (deployment focus)
    - Added backend-api-design.md (API patterns focus)
    - Maintained python-backend.md for Python specifics

---

## 💡 Contributing

When adding new languages:
1. Check if `universal-principles.md` applies
2. Check if `backend-api-design.md` or `frontend-web.md` applies
3. Create language-specific file only for unique conventions
4. Update this INDEX.md

**Example:** For Ruby on Rails, you'd use:
- `universal-principles.md` ✅
- `backend-api-design.md` ✅
- Create `ruby-rails.md` for Rails-specific patterns

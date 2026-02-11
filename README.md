# Development Rules - Multi-Language

Comprehensive engineering rules for consistent, high-quality development across all tech stacks.

---

## 📚 How to Use

**AI Adaptation Strategy:** AI will automatically apply the appropriate rules based on file extension and project context. You don't need to specify which rules to use.

**Rule Priority:**
1. **Universal Principles** — Always applied to all projects
2. **Language-Specific Rules** — Applied based on file type

---

## 📂 Rule Files

### 🌐 [Universal Principles](./universal-principles.md)
**Apply to:** All languages and projects  
**Covers:**
- Verification & testing protocol
- Confirmation & communication (moderate policy)
- Workflow patterns (Planning → Execution → Verification)
- Code organization philosophy
- Response format standards
- Judgment guidelines

**Key Principle:** Simple tasks auto-proceed, complex tasks get best practice recommendations first.

---

### 🐍 [Python Backend](./python-backend.md)
**Apply to:** `.py` files (FastAPI, Django, Flask)  
**Focus:** FastAPI, async/await, type safety, serverless deployment  
**Covers:**
- Strict typing (non-negotiable)
- Google-style docstrings
- Async/await patterns
- FastAPI best practices
- Pydantic validation
- Vercel/serverless deployment (<50MB limit)
- Error handling with Result pattern

**Priority:** Type safety and async operations

---

### 🎨 [Frontend Development](./frontend.md)
**Apply to:** `.html`, `.css`, `.js`, `.ts`, `.jsx`, `.tsx` files  
**Focus:** **Deployment-first**, modern design, component architecture  
**Covers:**
- Technology stack decisions (Vanilla → React → Next.js)
- Premium design aesthetics (NO boring interfaces!)
- CSS best practices (design system, component styles)
- Modern JavaScript/TypeScript (ES6+, strict mode)
- SEO & accessibility
- Performance optimization (Web Vitals)
- **Deployment strategies** (Vercel, optimization, CDN)

**Priority:** Deployment readiness and visual excellence

---

### ⚙️ [Backend Development](./backend.md)
**Apply to:** `.go`, `.php` files (also applies to Python for API design)  
**Focus:** **API Design Patterns**, RESTful best practices, scalability  
**Covers:**
- RESTful API principles (resource-based URLs, HTTP methods)
- Consistent response formats (success/error)
- HTTP status codes (correct usage)
- API versioning strategies
- Go best practices (handlers, middleware, error handling)
- PHP best practices (Laravel/Symfony patterns)
- Authentication & authorization (JWT, OAuth2)
- Rate limiting & caching
- Database optimization (connection pooling, N+1 prevention)

**Priority:** API design patterns and scalability

---

## 🎯 AI Decision Tree

```
File detected
    │
    ├─→ python-backend.md (if .py)
    ├─→ frontend.md (if .html/.css/.js/.ts)
    ├─→ backend.md (if .go/.php OR Python API design)
    │
    └─→ universal-principles.md (ALWAYS applied)
```

---

## 🚀 Quick Reference

### For Simple Tasks (Auto-Proceed)
AI will directly:
- Fix syntax errors, typos
- Add missing documentation
- Remove unused imports
- Create test scripts (after feature approved)

**No confirmation needed** — AI reports: "✅ Fixed [issue]. Syntax check passed."

### For Complex Tasks (Best Practice Recommendations)
AI will:
1. Analyze requirements
2. Research 2-3 viable options
3. Present pros/cons for each
4. **Recommend best practice** (marked clearly)
5. Ask: "Saya recommend Opsi 1 karena [reasons]. Mau proceed?"

**Wait for your approval** before executing.

---

## 📝 Example Scenarios

### Scenario 1: Python Backend Feature
```
USER: "Add user authentication endpoint"

AI: "Saya ada 2 opsi untuk authentication:

**Opsi 1 (Recommended): JWT dengan FastAPI security**
✅ Pro: Built-in FastAPI support
✅ Pro: Lightweight, no extra dependencies
✅ Pro: Industry standard
❌ Con: Need token refresh logic

**Opsi 2: OAuth2 provider (Auth0/Supabase)**
✅ Pro: Managed service, easier
❌ Con: Extra dependency (~15MB)
❌ Con: External service dependency

Rekomendasi: **Opsi 1** karena lightweight dan Vercel-friendly.
Mau proceed?"

[Wait for approval]
[Execute approved approach]
```

### Scenario 2: Frontend Deployment
```
USER: "Deploy this landing page"

AI: [Applies frontend.md rules]
1. Check optimization (minify, compress, images)
2. Verify SEO tags (meta, OG)
3. Test Web Vitals
4. Build production bundle
5. Deploy to Vercel
6. Report: "✅ Deployed to [URL]. LCP: 1.8s, perfect!"
```

### Scenario 3: Simple Bug Fix
```
USER: "Fix syntax error on line 42"

AI: [Applies universal-principles.md]
1. Analyze error
2. Fix directly (obvious issue)
3. Run syntax check
4. Report: "✅ Fixed missing import. Syntax check passed."

[No confirmation needed - simple task]
```

---

## 🔄 Updating Rules

**To modify rules:**
1. Edit the relevant `.md` file in this directory
2. AI will automatically use updated rules in next conversation

**To add new language:**
Create `rules/language-name.md` following the same structure.

---

## 💡 Philosophy

1. **Quality > Speed** — Take time to verify
2. **Best Practices First** — Always recommend industry standards
3. **Context-Aware** — Rules adapt to project and language
4. **User-Friendly** — Simple tasks auto-proceed, complex tasks get guidance
5. **Production-Ready** — Every commit should be deployable

---

## 📞 Need Help?

If AI behavior doesn't match expectations, check:
1. Are you working with the correct file type?
2. Does the task classify as simple or complex?
3. Is there a best practice recommendation missing?

Adjust the relevant rule file as needed.

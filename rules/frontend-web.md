# Frontend Web Development Rules (HTML/CSS/JavaScript/TypeScript)

**Priority Focus: Deployment-First Development**

---

## 1. Project Structure & Organization

### Standard Folder Structure
```
project/
├── src/
│   ├── components/     # Reusable UI components
│   ├── pages/          # Page-level components
│   ├── styles/         # CSS/SCSS files
│   ├── utils/          # Helper functions
│   ├── services/       # API calls
│   ├── hooks/          # Custom React/Vue hooks
│   └── assets/         # Images, fonts, icons
├── public/             # Static files
├── tests/              # Test files
├── dist/               # Build output
└── package.json
```

### File Naming Conventions
* **Components:** PascalCase (e.g., `UserProfile.tsx`, `NavBar.vue`)
* **Utilities:** camelCase (e.g., `formatDate.ts`, `apiClient.ts`)
* **Styles:** kebab-case (e.g., `user-profile.css`, `nav-bar.module.css`)
* **Pages:** kebab-case or PascalCase (e.g., `home.tsx` or `HomePage.tsx`)

---

## 2. TypeScript Strict Mode (When Using TS)

### tsconfig.json Requirements
```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictPropertyInitialization": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true
  }
}
```

### Type Everything
```typescript
// ❌ Bad
function fetchUser(id) {
  return fetch(`/api/users/${id}`).then(r => r.json())
}

// ✅ Good
interface User {
  id: string
  name: string
  email: string
}

async function fetchUser(id: string): Promise<User> {
  const response: Response = await fetch(`/api/users/${id}`)
  const user: User = await response.json()
  return user
}
```

---

## 3. Component Best Practices

### React/Vue Component Structure
```typescript
// ✅ Good React Component
import { useState, useEffect } from 'react'
import type { FC } from 'react'

interface UserCardProps {
  userId: string
  onDelete?: (id: string) => void
}

export const UserCard: FC<UserCardProps> = ({ userId, onDelete }) => {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState<boolean>(true)

  useEffect(() => {
    fetchUser(userId)
      .then(setUser)
      .finally(() => setLoading(false))
  }, [userId])

  if (loading) return <LoadingSpinner />
  if (!user) return <ErrorMessage />

  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      {onDelete && (
        <button onClick={() => onDelete(user.id)}>
          Delete
        </button>
      )}
    </div>
  )
}
```

### Component Rules
* **Single Responsibility:** One component = one purpose
* **Props Typing:** Always define prop interfaces
* **Default Props:** Use default values when appropriate
* **Events:** Use descriptive handler names (`onSubmit`, not `handleClick`)

---

## 4. CSS/Styling Standards

### CSS Organization
```css
/* ✅ Good: BEM Naming Convention */
.user-card { /* Block */
  padding: 1rem;
  border: 1px solid #ddd;
}

.user-card__title { /* Element */
  font-size: 1.5rem;
  font-weight: bold;
}

.user-card--featured { /* Modifier */
  border-color: #007bff;
  background: #f0f8ff;
}
```

### Style Preferences
* **CSS Modules:** For component-scoped styles
* **Tailwind CSS:** If using utility-first approach
* **CSS Variables:** For theming
    ```css
    :root {
      --primary-color: #007bff;
      --spacing-unit: 8px;
    }
    ```

### Responsive Design
```css
/* Mobile-first approach */
.container {
  width: 100%;
  padding: 1rem;
}

/* Tablet */
@media (min-width: 768px) {
  .container {
    max-width: 720px;
    margin: 0 auto;
  }
}

/* Desktop */
@media (min-width: 1024px) {
  .container {
    max-width: 960px;
  }
}
```

---

## 5. Testing Standards

### Testing Pyramid
1. **Unit Tests (70%):** Individual functions/components
2. **Integration Tests (20%):** Component interactions
3. **E2E Tests (10%):** Full user flows

### Jest/Vitest Example
```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { UserCard } from './UserCard'

describe('UserCard', () => {
  it('renders user information', () => {
    render(<UserCard userId="123" />)
    
    expect(screen.getByText('John Doe')).toBeInTheDocument()
    expect(screen.getByText('john@example.com')).toBeInTheDocument()
  })

  it('calls onDelete when delete button clicked', () => {
    const mockDelete = jest.fn()
    render(<UserCard userId="123" onDelete={mockDelete} />)
    
    fireEvent.click(screen.getByText('Delete'))
    
    expect(mockDelete).toHaveBeenCalledWith('123')
  })
})
```

### Test Requirements
* **Coverage:** Minimum 70% code coverage
* **Critical Paths:** 100% coverage for auth, payment, data mutations
* **Snapshot Tests:** For UI components stability

---

## 6. DEPLOYMENT (Priority Focus)

### Pre-Deployment Checklist
- ✅ All tests passed (`npm test`)
- ✅ Build successful (`npm run build`)
- ✅ No TypeScript errors (`npm run type-check`)
- ✅ No lint errors (`npm run lint`)
- ✅ Bundle size analyzed (< target size)
- ✅ Environment variables documented
- ✅ Error tracking configured (Sentry/LogRocket)

### Platform-Specific Guidelines

#### Vercel Deployment
```json
// vercel.json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "framework": "vite",
  "env": {
    "VITE_API_URL": "@api-url"
  }
}
```

**Vercel Best Practices:**
* Use Edge Functions for API routes
* Configure caching headers properly
* Enable automatic HTTPS
* Set up preview deployments for PRs

#### Netlify Deployment
```toml
# netlify.toml
[build]
  command = "npm run build"
  publish = "dist"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

**Netlify Best Practices:**
* Use Netlify Functions for serverless
* Configure `_redirects` for SPA routing
* Enable form handling if needed
* Set up deploy notifications

### Performance Optimization
* **Code Splitting:** Dynamic imports for routes
    ```typescript
    const Dashboard = lazy(() => import('./pages/Dashboard'))
    ```
* **Asset Optimization:**
    - Images: WebP format, lazy loading
    - Fonts: Subset, font-display: swap
    - Icons: SVG sprites or icon fonts
* **Bundle Size:**
    - Analyze with `webpack-bundle-analyzer`
    - Target: < 200KB initial bundle (gzipped)
    - Lazy load heavy libraries

### Environment Variables
```bash
# .env.example
VITE_API_URL=https://api.example.com
VITE_ANALYTICS_ID=GA-XXXXX
VITE_SENTRY_DSN=https://xxx@sentry.io/xxx
```

**Rules:**
* Never commit `.env` to git
* Always provide `.env.example`
* Validate env vars at build time
* Use different values for dev/staging/prod

---

## 7. Error Handling & Logging

### Client-Side Error Handling
```typescript
// ✅ Good: Centralized error handler
class APIError extends Error {
  constructor(
    public statusCode: number,
    message: string
  ) {
    super(message)
    this.name = 'APIError'
  }
}

async function apiCall<T>(endpoint: string): Promise<T> {
  try {
    const response = await fetch(endpoint)
    
    if (!response.ok) {
      throw new APIError(
        response.status,
        `API call failed: ${response.statusText}`
      )
    }
    
    return await response.json()
  } catch (error) {
    if (error instanceof APIError) {
      // Log to error tracking service
      console.error('API Error:', error.statusCode, error.message)
    }
    throw error
  }
}
```

### User-Friendly Error Messages
```typescript
// ❌ Bad
alert('Error: 500')

// ✅ Good
const ERROR_MESSAGES = {
  500: 'Server error. Please try again later.',
  404: 'Resource not found.',
  403: 'You do not have permission to access this.',
  401: 'Please log in to continue.'
}

function showError(statusCode: number): void {
  const message = ERROR_MESSAGES[statusCode] || 'An unexpected error occurred.'
  toast.error(message)
}
```

---

## 8. Security Best Practices

### XSS Prevention
* **Never use `dangerouslySetInnerHTML`** unless absolutely necessary
* **Sanitize user input:** Use DOMPurify
* **CSP Headers:** Configure Content Security Policy

### API Security
```typescript
// ✅ Good: Secure API calls
const API_BASE = process.env.VITE_API_URL

async function secureApiCall<T>(
  endpoint: string,
  token?: string
): Promise<T> {
  const headers: HeadersInit = {
    'Content-Type': 'application/json'
  }
  
  if (token) {
    headers['Authorization'] = `Bearer ${token}`
  }
  
  const response = await fetch(`${API_BASE}${endpoint}`, {
    headers,
    credentials: 'include' // Send cookies
  })
  
  return response.json()
}
```

### Secure Storage
* **Never store secrets in localStorage**
* **Use httpOnly cookies** for tokens
* **Encrypt sensitive data** before storage

---

## 9. Accessibility (a11y)

### ARIA Requirements
```html
<!-- ✅ Good: Accessible button -->
<button
  aria-label="Close modal"
  aria-pressed="false"
  onClick={handleClose}
>
  <CloseIcon aria-hidden="true" />
</button>

<!-- ✅ Good: Accessible form -->
<form>
  <label htmlFor="email">Email</label>
  <input
    id="email"
    type="email"
    aria-required="true"
    aria-invalid={hasError}
    aria-describedby="email-error"
  />
  {hasError && (
    <span id="email-error" role="alert">
      Invalid email format
    </span>
  )}
</form>
```

### Keyboard Navigation
* All interactive elements must be keyboard accessible
* Logical tab order
* Visible focus indicators

---

## 10. Git & Collaboration

### Branch Strategy
```
main          # Production-ready code
└─ develop    # Integration branch
   ├─ feature/user-auth
   ├─ feature/dashboard
   └─ bugfix/login-error
```

### Commit Messages
```bash
feat: add user authentication flow
fix: resolve modal close button not working
style: update button colors to match design
test: add tests for UserCard component
docs: update deployment instructions
```

### Pull Request Template
```markdown
## Changes
- Brief description of changes

## Testing
- [ ] Unit tests pass
- [ ] UI tested in Chrome/Firefox/Safari
- [ ] Mobile responsive tested
- [ ] Accessibility checked

## Screenshots
[Add screenshots for UI changes]
```

---

## Summary: Frontend Best Practices

1. **Type Everything** — Use TypeScript strict mode
2. **Component-Driven** — Single responsibility, reusable components
3. **Test Thoroughly** — 70% coverage minimum
4. **Optimize Performance** — Code splitting, lazy loading, < 200KB bundle
5. **Deploy Smart** — Use Vercel/Netlify, configure env vars, enable caching
6. **Handle Errors** — User-friendly messages, error tracking
7. **Secure By Default** — Sanitize inputs, CSP headers, secure storage
8. **Accessibility First** — ARIA labels, keyboard navigation
9. **Git Discipline** — Conventional commits, PR reviews

**Deployment Priority:** Every change should consider impact on build size, performance, and deployment.

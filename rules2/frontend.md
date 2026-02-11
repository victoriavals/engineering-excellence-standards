# Frontend Development Rules (HTML/CSS/JS/TS)

**Extends:** [Universal Principles](./universal-principles.md)  
**Focus:** Deployment-first, modern design, component architecture

---

## 1. Technology Stack Priority

### Core Technologies
* **HTML5:** Semantic markup, accessibility
* **CSS3:** Modern styling (flexbox, grid, custom properties)
* **JavaScript (ES6+):** Vanilla JS for simple projects
* **TypeScript:** For complex applications (strict mode)

### Framework Decisions
* **Simple Sites:** Vanilla HTML/CSS/JS
* **SPAs:** React, Vue, Svelte
* **Full-Stack:** Next.js (React), Nuxt (Vue), SvelteKit
* **Avoid TailwindCSS** unless user explicitly requests

### Always Confirm Framework Choice
```markdown
Saya ada 2 opsi untuk project ini:

**Opsi 1 (Recommended): Vanilla HTML/CSS/JS**
✅ Pro: Lightweight, zero dependencies
✅ Pro: Fast load time, easy deployment
✅ Pro: No build step needed
❌ Con: Manual state management

**Opsi 2: Next.js/Vite**
✅ Pro: Component-based, scalable
✅ Pro: Hot reload, TypeScript support
❌ Con: Build complexity, larger bundle

Mau proceed dengan Opsi 1?
```

---

## 2. Project Structure

### Standard Organization
```
frontend/
├─ index.html
├─ styles/
│  ├─ index.css         # Design system, tokens
│  ├─ components.css    # Reusable components
│  └─ pages.css         # Page-specific styles
├─ scripts/
│  ├─ main.js           # Entry point
│  ├─ components/       # Component logic
│  └─ utils/            # Helpers, API calls
├─ assets/
│  ├─ images/
│  └─ icons/
└─ public/              # Static files
```

### Framework Projects (Next.js/Vite)
```
project/
├─ src/
│  ├─ components/       # Reusable UI components
│  ├─ pages/            # Route pages
│  ├─ styles/           # CSS modules/Tailwind
│  ├─ lib/              # Utilities, API clients
│  └─ types/            # TypeScript definitions
├─ public/              # Static assets
└─ tests/               # Component tests
```

---

## 3. Design Aesthetics (CRITICAL)

### Premium First Impression
**Goal:** User should be WOW-ed at first glance

❌ **Avoid:**
* Plain colors (basic red, blue, green)
* Default browser fonts
* Static, lifeless interfaces
* Generic templates

✅ **Use:**
* Curated color palettes (HSL-based, harmonious gradients)
* Modern typography (Google Fonts: Inter, Roboto, Outfit)
* Smooth micro-animations (hover effects, transitions)
* Dynamic, responsive design
* Glassmorphism, subtle shadows

### Visual Excellence Checklist
- ✅ Custom color palette (not generic colors)
- ✅ Modern typography (web fonts, not defaults)
- ✅ Smooth animations (transition: all 0.3s ease)
- ✅ Responsive design (mobile-first)
- ✅ Dark mode support (if applicable)
- ✅ Accessibility (ARIA labels, keyboard navigation)

### Example: Button Design
```css
/* ❌ BAD: Plain, boring */
.button {
    background: blue;
    color: white;
    padding: 10px;
}

/* ✅ GOOD: Premium, modern */
.button {
    background: linear-gradient(135deg, hsl(220, 80%, 55%), hsl(250, 70%, 60%));
    color: white;
    padding: 12px 24px;
    border-radius: 8px;
    font-family: 'Inter', sans-serif;
    font-weight: 600;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.button:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.2);
}
```

---

## 4. CSS Best Practices

### Design System First
```css
/* index.css - Define design tokens */
:root {
    /* Colors */
    --color-primary: hsl(220, 80%, 55%);
    --color-primary-dark: hsl(220, 80%, 45%);
    --color-secondary: hsl(280, 70%, 60%);
    --color-bg: hsl(0, 0%, 98%);
    --color-text: hsl(0, 0%, 10%);
    
    /* Spacing */
    --spacing-xs: 4px;
    --spacing-sm: 8px;
    --spacing-md: 16px;
    --spacing-lg: 24px;
    --spacing-xl: 32px;
    
    /* Typography */
    --font-sans: 'Inter', -apple-system, sans-serif;
    --font-heading: 'Outfit', sans-serif;
    --font-size-sm: 0.875rem;
    --font-size-base: 1rem;
    --font-size-lg: 1.25rem;
    --font-size-xl: 1.5rem;
    
    /* Effects */
    --shadow-sm: 0 2px 8px rgba(0, 0, 0, 0.1);
    --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.15);
    --shadow-lg: 0 8px 24px rgba(0, 0, 0, 0.2);
    --transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}
```

### Component-Based Styles
```css
/* components.css */
.card {
    background: white;
    border-radius: 12px;
    padding: var(--spacing-lg);
    box-shadow: var(--shadow-md);
    transition: var(--transition);
}

.card:hover {
    transform: translateY(-4px);
    box-shadow: var(--shadow-lg);
}
```

### Avoid Inline Styles
```html
<!-- ❌ BAD -->
<div style="color: red; padding: 10px;">Content</div>

<!-- ✅ GOOD -->
<div class="alert alert--error">Content</div>
```

---

## 5. JavaScript/TypeScript Standards

### Modern JavaScript (ES6+)
```javascript
// ❌ BAD: Old syntax
var data = [];
function fetchData(url, callback) {
    var xhr = new XMLHttpRequest();
    // ...
}

// ✅ GOOD: Modern syntax
const data = [];
async function fetchData(url) {
    try {
        const response = await fetch(url);
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Fetch error:', error);
        return null;
    }
}
```

### TypeScript (Strict Mode)
```typescript
// tsconfig.json
{
    "compilerOptions": {
        "strict": true,
        "noImplicitAny": true,
        "strictNullChecks": true
    }
}

// ❌ BAD: No types
function processUser(user) {
    return user.name;
}

// ✅ GOOD: Explicit types
interface User {
    id: string;
    name: string;
    email: string;
}

function processUser(user: User): string {
    return user.name;
}
```

### Component Pattern (Vanilla JS)
```javascript
/**
 * Create a reusable card component.
 * 
 * @param {Object} data - Card data
 * @param {string} data.title - Card title
 * @param {string} data.description - Card description
 * @returns {HTMLElement} Card element
 */
function createCard({ title, description }) {
    const card = document.createElement('div');
    card.className = 'card';
    
    card.innerHTML = `
        <h3 class="card__title">${title}</h3>
        <p class="card__description">${description}</p>
    `;
    
    return card;
}
```

---

## 6. Deployment Focus (CRITICAL)

### Build for Production
```json
// package.json
{
    "scripts": {
        "dev": "vite",
        "build": "vite build",
        "preview": "vite preview",
        "deploy": "npm run build && vercel --prod"
    }
}
```

### Optimization Checklist
- ✅ Minify CSS/JS (automatic with Vite/Next.js)
- ✅ Optimize images (WebP format, lazy loading)
- ✅ Code splitting (dynamic imports)
- ✅ Remove unused code (tree shaking)
- ✅ Compress assets (gzip/brotli)
- ✅ CDN for static files

### Vercel Deployment
```bash
# Initialize with non-interactive mode
npx -y create-vite@latest ./ --template vanilla

# Or Next.js
npx -y create-next-app@latest ./ --typescript --tailwind --app

# Deploy
vercel --prod
```

### Environment Variables
```javascript
// ❌ BAD: Hardcoded
const API_URL = "https://api.example.com";

// ✅ GOOD: Environment-based
const API_URL = import.meta.env.VITE_API_URL || 
                process.env.NEXT_PUBLIC_API_URL;
```

---

## 7. SEO & Accessibility

### HTML Best Practices
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- SEO -->
    <title>Page Title - Site Name</title>
    <meta name="description" content="Concise page description (150-160 chars)">
    
    <!-- Open Graph -->
    <meta property="og:title" content="Page Title">
    <meta property="og:description" content="Description">
    <meta property="og:image" content="/og-image.jpg">
    
    <!-- Performance -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
</head>
<body>
    <!-- Semantic HTML -->
    <header>
        <nav aria-label="Main navigation">
            <!-- Navigation -->
        </nav>
    </header>
    
    <main>
        <h1>Single H1 per page</h1>
        <article>
            <h2>Section heading</h2>
            <!-- Content -->
        </article>
    </main>
    
    <footer>
        <!-- Footer content -->
    </footer>
</body>
</html>
```

### Accessibility
* Use semantic HTML (`<nav>`, `<main>`, `<article>`)
* ARIA labels for interactive elements
* Keyboard navigation support (tab order)
* Color contrast ratio ≥ 4.5:1
* Alt text for images

---

## 8. Testing

### Test Structure (Frontend)
```javascript
// Using Vitest or Jest
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import { Button } from './Button';

describe('Button Component', () => {
    it('renders button with text', () => {
        render(<Button>Click me</Button>);
        expect(screen.getByText('Click me')).toBeInTheDocument();
    });
    
    it('handles click events', async () => {
        const handleClick = vi.fn();
        render(<Button onClick={handleClick}>Click</Button>);
        
        await userEvent.click(screen.getByText('Click'));
        expect(handleClick).toHaveBeenCalledOnce();
    });
    
    it('applies custom className', () => {
        render(<Button className="custom">Button</Button>);
        expect(screen.getByText('Button')).toHaveClass('custom');
    });
});
```

### Browser Testing Checklist
- ✅ Chrome/Edge (latest)
- ✅ Firefox (latest)
- ✅ Safari (latest)
- ✅ Mobile browsers (iOS Safari, Chrome Android)
- ✅ Responsive breakpoints (mobile, tablet, desktop)

---

## 9. Performance

### Web Vitals Targets
* **LCP (Largest Contentful Paint):** < 2.5s
* **FID (First Input Delay):** < 100ms
* **CLS (Cumulative Layout Shift):** < 0.1

### Optimization Techniques
```javascript
// Lazy loading images
<img src="image.jpg" loading="lazy" alt="Description">

// Dynamic imports (code splitting)
const HeavyComponent = lazy(() => import('./HeavyComponent'));

// Debounce expensive operations
function debounce(func, wait) {
    let timeout;
    return function executedFunction(...args) {
        clearTimeout(timeout);
        timeout = setTimeout(() => func(...args), wait);
    };
}

const searchHandler = debounce((query) => {
    // Expensive search operation
}, 300);
```

---

## Summary: Frontend Checklist

✅ **Deployment First** — Build process, optimization, Vercel ready  
✅ **Premium Design** — Modern aesthetics, animations, custom palettes  
✅ **Component Architecture** — Reusable, organized, maintainable  
✅ **TypeScript Strict Mode** — Type safety for complex apps  
✅ **SEO & Accessibility** — Meta tags, semantic HTML, ARIA  
✅ **Performance Optimized** — Lazy loading, code splitting, Web Vitals  
✅ **Responsive Design** — Mobile-first, all breakpoints  
✅ **Testing Coverage** — Component tests, browser testing  
✅ **Environment Configuration** — No hardcoded values  
✅ **Modern Tooling** — Vite/Next.js, ESLint, Prettier

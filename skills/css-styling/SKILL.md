---
name: css-styling
description: Provides scalable, maintainable CSS architecture with responsive design, color scheme support, and performance-conscious animations. Covers CSS Modules, mobile-first responsive design, light/dark mode, layout patterns, logical properties, and typography. Use when styling, writing CSS/SCSS, implementing responsive layouts, media queries, dark mode, or animations.
compatibility: Designed for Claude Code. Uses chrome-devtools MCP for browser verification.
---

# CSS Architecture & Responsive Design

This skill provides patterns for scalable, maintainable CSS with responsive design, color scheme support, and performance-conscious animations. Framework-agnostic, with CSS Modules as the preferred scoping strategy.

## CSS Modules (Preferred)

CSS Modules provide component-scoped styles without runtime overhead:

```css
/* Button.module.css */
.root {
  display: inline-flex;
  align-items: center;
  gap: var(--space-2);
  padding: var(--space-2) var(--space-4);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  background: var(--color-surface);
  color: var(--color-text);
  font: inherit;
  cursor: pointer;
}

.root:hover {
  background: var(--color-surface-hover);
}

.root:focus-visible {
  outline: 2px solid var(--color-focus);
  outline-offset: 2px;
}

.primary {
  background: var(--color-primary);
  color: var(--color-on-primary);
  border-color: transparent;
}
```

**Rules:**
- One `.module.css` file per component, co-located with the component
- Use `.root` for the component's outermost element
- Compose variants as separate classes (`.primary`, `.small`, `.disabled`)
- Reference design tokens via CSS custom properties — never hardcode colors, spacing, or sizes
- Use `composes` for sharing styles between CSS Module files when needed

See `references/css-modules-patterns.md` for advanced patterns.

## Mobile-First Responsive Design

Always write base styles for the smallest viewport, then layer on complexity:

```css
/* Base: mobile (no media query) */
.grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: var(--space-4);
}

/* Tablet and up */
@media (min-width: 768px) {
  .grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Desktop and up */
@media (min-width: 1024px) {
  .grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

**Breakpoints (min-width):**
| Name | Value | Target |
|---|---|---|
| `sm` | `640px` | Large phones |
| `md` | `768px` | Tablets |
| `lg` | `1024px` | Laptops |
| `xl` | `1280px` | Desktops |

**Rules:**
- Always use `min-width` (mobile-first), never `max-width` as the primary approach
- Don't target specific devices — use content-driven breakpoints when standard ones don't fit
- Test at 320px minimum (WCAG reflow requirement)
- Content must reflow to single column at narrow widths — no horizontal scrolling

## Progressive Enhancement with @supports

Use `@supports` to layer modern CSS while keeping fallbacks:

```css
.layout {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

@supports (display: grid) {
  .layout {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  }
}

@supports (container-type: inline-size) {
  .card-container {
    container-type: inline-size;
  }

  @container (min-width: 400px) {
    .card {
      flex-direction: row;
    }
  }
}
```

## Color Scheme: Light/Dark Mode

Use CSS custom properties as design tokens, switched via `prefers-color-scheme`:

```css
:root {
  /* Light theme (default) */
  --color-bg: #ffffff;
  --color-surface: #f8f9fa;
  --color-surface-hover: #e9ecef;
  --color-text: #212529;
  --color-text-muted: #6c757d;
  --color-border: #dee2e6;
  --color-primary: #0d6efd;
  --color-on-primary: #ffffff;
  --color-focus: #0d6efd;
  --color-error: #dc3545;
  --color-success: #198754;

  color-scheme: light dark;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #1a1a2e;
    --color-surface: #16213e;
    --color-surface-hover: #1a2744;
    --color-text: #e2e8f0;
    --color-text-muted: #94a3b8;
    --color-border: #334155;
    --color-primary: #60a5fa;
    --color-on-primary: #1e293b;
    --color-focus: #60a5fa;
    --color-error: #f87171;
    --color-success: #34d399;
  }
}
```

**Rules:**
- All colors referenced as `var(--color-*)` — never hardcode hex/rgb in components
- Use `color-scheme: light dark` on `:root` to opt into UA dark mode defaults
- Test contrast ratios in BOTH themes (defer to `a11y` skill for specific ratios)
- Shadows, images, and borders may need adjustment per theme

See `references/color-scheme-tokens.md` for the full token set.

## Spacing Scale

Use a consistent spacing scale via custom properties:

```css
:root {
  --space-0: 0;
  --space-1: 0.25rem;  /* 4px */
  --space-2: 0.5rem;   /* 8px */
  --space-3: 0.75rem;  /* 12px */
  --space-4: 1rem;     /* 16px */
  --space-5: 1.25rem;  /* 20px */
  --space-6: 1.5rem;   /* 24px */
  --space-8: 2rem;     /* 32px */
  --space-10: 2.5rem;  /* 40px */
  --space-12: 3rem;    /* 48px */
  --space-16: 4rem;    /* 64px */
}
```

Use spacing tokens for all padding, margin, and gap values.

## Typography

```css
:root {
  --font-sans: system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif;
  --font-mono: ui-monospace, 'Cascadia Code', 'Fira Code', monospace;

  --text-xs: 0.75rem;    /* 12px */
  --text-sm: 0.875rem;   /* 14px */
  --text-base: 1rem;     /* 16px */
  --text-lg: 1.125rem;   /* 18px */
  --text-xl: 1.25rem;    /* 20px */
  --text-2xl: 1.5rem;    /* 24px */
  --text-3xl: 1.875rem;  /* 30px */

  --leading-tight: 1.25;
  --leading-normal: 1.5;
  --leading-relaxed: 1.75;

  --font-weight-normal: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;
}
```

**Rules:**
- Base font size: `16px` minimum (never smaller for body text)
- Line height: `1.5` for body text (WCAG text spacing)
- Use `rem` for font sizes — allows user zoom
- Limit line length to ~65-75 characters with `max-width: 65ch`

## Layout: Grid & Flexbox

**When to use which:**
| Use Case | Tool |
|---|---|
| Page-level layout | CSS Grid |
| Card grids / galleries | CSS Grid with `auto-fit`/`auto-fill` |
| Navigation bars | Flexbox |
| Centering | Flexbox or Grid `place-items: center` |
| Sidebar + content | Grid with `grid-template-columns` |
| Inline elements with spacing | Flexbox with `gap` |
| Overlapping elements | Grid with overlapping grid areas |

**Common patterns:**
```css
/* Responsive grid that auto-wraps */
.auto-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(250px, 100%), 1fr));
  gap: var(--space-4);
}

/* Sidebar layout */
.sidebar-layout {
  display: grid;
  grid-template-columns: minmax(200px, 300px) 1fr;
  gap: var(--space-6);
}

/* Stack (vertical flex) */
.stack {
  display: flex;
  flex-direction: column;
  gap: var(--space-4);
}

/* Cluster (horizontal flex, wraps) */
.cluster {
  display: flex;
  flex-wrap: wrap;
  gap: var(--space-2);
  align-items: center;
}
```

## Logical Properties

Use logical properties for internationalization support:

| Physical | Logical |
|---|---|
| `margin-left` | `margin-inline-start` |
| `margin-right` | `margin-inline-end` |
| `padding-top` | `padding-block-start` |
| `padding-bottom` | `padding-block-end` |
| `width` | `inline-size` |
| `height` | `block-size` |
| `text-align: left` | `text-align: start` |
| `border-left` | `border-inline-start` |

Use logical properties by default for new code. They work correctly for both LTR and RTL layouts.

## Animation & Transitions

**Performance rules:**
- Only animate `transform` and `opacity` — they don't trigger layout or paint
- Use `will-change` sparingly and only on elements about to animate
- Prefer CSS transitions for simple state changes, CSS animations for complex sequences
- Always respect `prefers-reduced-motion`

```css
.fade-in {
  opacity: 0;
  transform: translateY(8px);
  transition: opacity 200ms ease-out, transform 200ms ease-out;
}

.fade-in.visible {
  opacity: 1;
  transform: translateY(0);
}

@media (prefers-reduced-motion: reduce) {
  .fade-in {
    transition: none;
    opacity: 1;
    transform: none;
  }
}
```

**Timing guidelines:**
- Micro-interactions (hover, focus): 100-150ms
- Reveals and transitions: 200-300ms
- Complex animations: 300-500ms max
- Use `ease-out` for entrances, `ease-in` for exits, `ease-in-out` for ongoing

## Radius & Shadows

```css
:root {
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 0.75rem;
  --radius-xl: 1rem;
  --radius-full: 9999px;

  --shadow-sm: 0 1px 2px rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px rgb(0 0 0 / 0.1);
}
```

## SCSS (For Existing Projects)

When a project already uses SCSS, follow the same principles but leverage SCSS features:
- Use SCSS variables as build-time tokens, CSS custom properties as runtime tokens
- Prefer `@use` and `@forward` over `@import`
- Limit nesting to 2-3 levels max — deep nesting creates specificity issues
- Use mixins for responsive patterns, not for single-property utilities

## Browser Verification with MCP Tools

When verifying styles in the browser, use chrome-devtools MCP tools:

### Visual Verification
Use `take_screenshot` to verify:
- Layout renders correctly at current viewport
- Typography, spacing, and alignment match design
- Colors and contrast are correct

### Responsive Testing
Use `emulate` with viewport sizes:
- `{ width: 320, height: 568 }` — small mobile
- `{ width: 768, height: 1024 }` — tablet
- `{ width: 1024, height: 768 }` — laptop
- `{ width: 1280, height: 800 }` — desktop

### Color Scheme Testing
Use `emulate` to toggle:
- `colorScheme: "dark"` — verify dark mode tokens apply correctly
- `colorScheme: "light"` — verify light mode

### Element Inspection
Use `take_snapshot` to check:
- Elements have expected dimensions and positioning
- Computed styles match expectations

## Code Review Rules

When reviewing CSS code, flag:
1. **Hardcoded colors** — should use `var(--color-*)` tokens
2. **Hardcoded spacing** — should use `var(--space-*)` tokens
3. **`max-width` media queries** — should be mobile-first `min-width`
4. **Missing reduced-motion** — animations without `prefers-reduced-motion` fallback
5. **`outline: none`** without focus replacement — defer to `a11y` skill
6. **Deep nesting** — more than 3 levels of selector nesting
7. **`!important`** — almost always a sign of specificity problems
8. **`px` for font sizes** — should use `rem`
9. **Physical properties** in new code — should use logical properties
10. **Animating layout properties** — `width`, `height`, `top`, `left` cause jank

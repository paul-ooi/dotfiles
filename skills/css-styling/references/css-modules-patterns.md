# CSS Modules Patterns

## Contents
- File Naming & Co-location
- Basic Usage
- Composition with `composes`
- Conditional Classes Pattern
- Global Styles & Tokens
- Responsive Styles Within Modules
- Dark Mode Within Modules
- Animation Within Modules
- Anti-Patterns

## File Naming & Co-location

```
components/
  Button/
    Button.tsx
    Button.module.css
    Button.test.tsx
  Card/
    Card.tsx
    Card.module.css
```

One module file per component. Name matches the component.

## Basic Usage

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

/* Variants */
.primary {
  background: var(--color-primary);
  color: var(--color-on-primary);
  border-color: transparent;
}

.secondary {
  background: transparent;
  color: var(--color-primary);
  border-color: var(--color-primary);
}

/* Sizes */
.small {
  padding: var(--space-1) var(--space-2);
  font-size: var(--text-sm);
}

.large {
  padding: var(--space-3) var(--space-6);
  font-size: var(--text-lg);
}

/* States */
.root:hover {
  background: var(--color-surface-hover);
}

.root:focus-visible {
  outline: 2px solid var(--color-focus);
  outline-offset: 2px;
}

.root:disabled,
.root[aria-disabled="true"] {
  opacity: 0.5;
  cursor: not-allowed;
}
```

## Composition with `composes`

Share styles between CSS Module files without duplication:

```css
/* shared/typography.module.css */
.heading {
  font-weight: var(--font-weight-semibold);
  line-height: var(--leading-tight);
  color: var(--color-text);
}

.body {
  font-weight: var(--font-weight-normal);
  line-height: var(--leading-normal);
  color: var(--color-text);
}
```

```css
/* Card.module.css */
.title {
  composes: heading from '../shared/typography.module.css';
  font-size: var(--text-xl);
  margin-block-end: var(--space-2);
}
```

## Conditional Classes Pattern

Combine classes based on props (framework-agnostic concept):

```javascript
// Conceptual — works with any framework
const classNames = [
  styles.root,
  variant === 'primary' && styles.primary,
  variant === 'secondary' && styles.secondary,
  size === 'small' && styles.small,
  size === 'large' && styles.large,
  fullWidth && styles.fullWidth,
  className, // Allow consumer to add classes
].filter(Boolean).join(' ');
```

For utilities like `clsx` or `classnames`:
```javascript
import clsx from 'clsx';

const cls = clsx(
  styles.root,
  styles[variant],
  styles[size],
  { [styles.fullWidth]: fullWidth },
  className
);
```

## Global Styles & Tokens

Keep global styles minimal — tokens and resets only:

```css
/* globals.css (not a module) */
*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: var(--font-sans);
  font-size: var(--text-base);
  line-height: var(--leading-normal);
  color: var(--color-text);
  background: var(--color-bg);
  -webkit-font-smoothing: antialiased;
}

/* tokens.css (not a module) */
:root {
  /* Import or define all design tokens here */
}
```

## Responsive Styles Within Modules

```css
/* Card.module.css */
.root {
  display: grid;
  gap: var(--space-4);
  padding: var(--space-4);
}

.imageWrapper {
  aspect-ratio: 16 / 9;
  overflow: hidden;
  border-radius: var(--radius-md);
}

/* Stack on mobile, side-by-side on tablet+ */
@media (min-width: 768px) {
  .root {
    grid-template-columns: 300px 1fr;
  }

  .imageWrapper {
    aspect-ratio: 1;
  }
}
```

## Dark Mode Within Modules

Components inherit dark mode from CSS custom properties. No per-component dark mode logic needed:

```css
/* This just works — tokens switch automatically */
.card {
  background: var(--color-surface);
  color: var(--color-text);
  border: 1px solid var(--color-border);
}
```

For component-specific dark mode adjustments (rare):
```css
.card {
  background: var(--color-surface);
  box-shadow: var(--shadow-md);
}

@media (prefers-color-scheme: dark) {
  .card {
    box-shadow: none;
    border: 1px solid var(--color-border);
  }
}
```

## Animation Within Modules

```css
.toast {
  transform: translateY(100%);
  opacity: 0;
  transition: transform 200ms ease-out, opacity 200ms ease-out;
}

.toast.visible {
  transform: translateY(0);
  opacity: 1;
}

@media (prefers-reduced-motion: reduce) {
  .toast {
    transition: none;
  }

  .toast.visible {
    transform: none;
    opacity: 1;
  }
}
```

## Anti-Patterns

**Don't:**
- Use `:global()` to override other component styles — breaks encapsulation
- Create utility class modules (`.flex`, `.mt-4`) — use tokens in component styles instead
- Nest selectors deeply — keep specificity flat
- Use element selectors (`div`, `span`) — use class names for all styled elements
- Reference another component's internal class names — use composition or wrapper styles

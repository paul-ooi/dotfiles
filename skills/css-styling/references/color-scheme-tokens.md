# Color Scheme Tokens

Complete token set for light and dark color schemes. All component styles should reference these tokens via `var(--color-*)` — never hardcode color values.

## Core Token Definitions

```css
:root {
  /* === Color Scheme === */
  color-scheme: light dark;

  /* === Light Theme (Default) === */

  /* Backgrounds */
  --color-bg: #ffffff;
  --color-surface: #f8f9fa;
  --color-surface-hover: #e9ecef;
  --color-surface-active: #dee2e6;
  --color-surface-raised: #ffffff;
  --color-overlay: rgb(0 0 0 / 0.5);

  /* Text */
  --color-text: #212529;
  --color-text-muted: #6c757d;
  --color-text-inverse: #ffffff;

  /* Borders */
  --color-border: #dee2e6;
  --color-border-strong: #adb5bd;
  --color-border-subtle: #f1f3f5;

  /* Primary */
  --color-primary: #0d6efd;
  --color-primary-hover: #0b5ed7;
  --color-primary-active: #0a58ca;
  --color-on-primary: #ffffff;
  --color-primary-subtle: #cfe2ff;
  --color-on-primary-subtle: #084298;

  /* Semantic: Error */
  --color-error: #dc3545;
  --color-error-hover: #bb2d3b;
  --color-on-error: #ffffff;
  --color-error-subtle: #f8d7da;
  --color-on-error-subtle: #842029;

  /* Semantic: Warning */
  --color-warning: #ffc107;
  --color-warning-hover: #ffca2c;
  --color-on-warning: #212529;
  --color-warning-subtle: #fff3cd;
  --color-on-warning-subtle: #664d03;

  /* Semantic: Success */
  --color-success: #198754;
  --color-success-hover: #157347;
  --color-on-success: #ffffff;
  --color-success-subtle: #d1e7dd;
  --color-on-success-subtle: #0f5132;

  /* Semantic: Info */
  --color-info: #0dcaf0;
  --color-info-hover: #31d2f2;
  --color-on-info: #212529;
  --color-info-subtle: #cff4fc;
  --color-on-info-subtle: #055160;

  /* Focus */
  --color-focus: #0d6efd;
  --color-focus-ring: rgb(13 110 253 / 0.25);

  /* Shadows */
  --shadow-sm: 0 1px 2px rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
  --shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1);
}

/* === Dark Theme === */
@media (prefers-color-scheme: dark) {
  :root {
    /* Backgrounds */
    --color-bg: #0f172a;
    --color-surface: #1e293b;
    --color-surface-hover: #2d3a4f;
    --color-surface-active: #3b4a63;
    --color-surface-raised: #1e293b;
    --color-overlay: rgb(0 0 0 / 0.7);

    /* Text */
    --color-text: #e2e8f0;
    --color-text-muted: #94a3b8;
    --color-text-inverse: #0f172a;

    /* Borders */
    --color-border: #334155;
    --color-border-strong: #475569;
    --color-border-subtle: #1e293b;

    /* Primary */
    --color-primary: #60a5fa;
    --color-primary-hover: #93bbfd;
    --color-primary-active: #3b82f6;
    --color-on-primary: #1e293b;
    --color-primary-subtle: #1e3a5f;
    --color-on-primary-subtle: #93c5fd;

    /* Semantic: Error */
    --color-error: #f87171;
    --color-error-hover: #fca5a5;
    --color-on-error: #1e293b;
    --color-error-subtle: #5f2120;
    --color-on-error-subtle: #fecaca;

    /* Semantic: Warning */
    --color-warning: #fbbf24;
    --color-warning-hover: #fcd34d;
    --color-on-warning: #1e293b;
    --color-warning-subtle: #5f4d07;
    --color-on-warning-subtle: #fde68a;

    /* Semantic: Success */
    --color-success: #34d399;
    --color-success-hover: #6ee7b7;
    --color-on-success: #1e293b;
    --color-success-subtle: #14532d;
    --color-on-success-subtle: #a7f3d0;

    /* Semantic: Info */
    --color-info: #22d3ee;
    --color-info-hover: #67e8f9;
    --color-on-info: #1e293b;
    --color-info-subtle: #164e63;
    --color-on-info-subtle: #a5f3fc;

    /* Focus */
    --color-focus: #60a5fa;
    --color-focus-ring: rgb(96 165 250 / 0.3);

    /* Shadows — more subtle in dark mode */
    --shadow-sm: 0 1px 2px rgb(0 0 0 / 0.2);
    --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.3), 0 2px 4px -2px rgb(0 0 0 / 0.3);
    --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.3), 0 4px 6px -4px rgb(0 0 0 / 0.3);
    --shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.3), 0 8px 10px -6px rgb(0 0 0 / 0.3);
  }
}
```

## Token Naming Convention

Tokens follow a semantic naming pattern:

```
--color-{category}
--color-{category}-{variant}
--color-on-{category}          /* text color for use ON that category background */
--color-{category}-subtle      /* muted background */
--color-on-{category}-subtle   /* text for use on the subtle background */
```

**Categories:** `bg`, `surface`, `text`, `border`, `primary`, `error`, `warning`, `success`, `info`, `focus`
**Variants:** `hover`, `active`, `strong`, `subtle`, `inverse`

## Usage in Components

```css
/* Always use tokens, never raw colors */

/* Good */
.alert {
  background: var(--color-error-subtle);
  color: var(--color-on-error-subtle);
  border: 1px solid var(--color-error);
}

/* Bad — hardcoded colors */
.alert {
  background: #f8d7da;
  color: #842029;
  border: 1px solid #dc3545;
}
```

## Extending Tokens

To add a new semantic color (e.g., for a specific feature):

```css
:root {
  --color-accent: #8b5cf6;
  --color-accent-hover: #7c3aed;
  --color-on-accent: #ffffff;
  --color-accent-subtle: #ede9fe;
  --color-on-accent-subtle: #5b21b6;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-accent: #a78bfa;
    --color-accent-hover: #c4b5fd;
    --color-on-accent: #1e293b;
    --color-accent-subtle: #3b1f7a;
    --color-on-accent-subtle: #ddd6fe;
  }
}
```

## Contrast Requirements

All token pairs must meet WCAG AA contrast ratios (defer to `a11y` skill for specifics):
- `--color-text` on `--color-bg`: >= 4.5:1
- `--color-text` on `--color-surface`: >= 4.5:1
- `--color-text-muted` on `--color-bg`: >= 4.5:1
- `--color-on-primary` on `--color-primary`: >= 4.5:1
- `--color-on-error` on `--color-error`: >= 4.5:1
- All `--color-on-*-subtle` on their `--color-*-subtle`: >= 4.5:1

Test in both light AND dark themes.

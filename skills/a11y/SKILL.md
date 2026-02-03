---
triggers:
  - accessibility
  - a11y
  - WCAG
  - ARIA
  - screen reader
  - keyboard navigation
  - focus management
  - alt text
---

# Accessibility Skill — WCAG 2.2 AA Compliance

This skill provides rules and patterns for building accessible web interfaces that conform to WCAG 2.2 Level AA. All guidance is framework-agnostic and applies to any HTML-based UI.

## Foundational Principle: Native HTML First, ARIA Last

**This is the single most important rule in this skill.** Always exhaust native HTML options before introducing any ARIA attributes.

Native HTML elements provide keyboard behavior, focus management, form participation, validation, and screen reader semantics **for free**. ARIA provides none of these — it only changes what assistive technology *announces*, not how the element *behaves*. Every ARIA attribute you add is code you must maintain, test, and keep in sync with actual behavior.

### The decision process for every interactive element:

1. **Is there a native HTML element that does this?** Use it. (`<button>`, `<a>`, `<select>`, `<details>`, `<dialog>`, `<input>`, `<progress>`, `<meter>`, etc.)
2. **Can native elements be combined to achieve this?** Use them. (`<fieldset>` + `<legend>`, `<label>` + `<input>`, `<details>` + `<summary>`, `<table>` + `<th>`)
3. **Can the native element be styled to match the design?** Almost always yes — style the native element rather than rebuilding it.
4. **Only if no native HTML combination works** — then use ARIA, and you MUST also implement all keyboard behavior, focus management, and state changes yourself.

### Common native HTML elements to use INSTEAD of ARIA:

| Need | Use native HTML | NOT this |
|---|---|---|
| Clickable action | `<button>` | `<div role="button" tabindex="0">` |
| Navigation link | `<a href="...">` | `<span role="link" tabindex="0">` |
| Checkbox | `<input type="checkbox">` | `<div role="checkbox" aria-checked>` |
| Radio buttons | `<input type="radio">` in `<fieldset>` | `<div role="radiogroup">` + `<div role="radio">` |
| Dropdown selection | `<select>` | `<div role="listbox">` |
| Text input | `<input>` / `<textarea>` | `<div role="textbox" contenteditable>` |
| Show/hide content | `<details>` + `<summary>` | `<div>` + `aria-expanded` |
| Modal dialog | `<dialog>` with `.showModal()` | `<div role="dialog" aria-modal>` |
| Progress indicator | `<progress>` | `<div role="progressbar">` |
| Meter / gauge | `<meter>` | `<div role="meter">` |
| Data table | `<table>` + `<th>` + `<caption>` | `<div role="table">` |
| Form grouping | `<fieldset>` + `<legend>` | `<div role="group" aria-label>` |
| Figure with caption | `<figure>` + `<figcaption>` | `<div role="figure" aria-label>` |
| Output / result | `<output>` | `<span role="status">` |
| Navigation | `<nav>` | `<div role="navigation">` |
| Page header | `<header>` | `<div role="banner">` |
| Page footer | `<footer>` | `<div role="contentinfo">` |
| Sidebar | `<aside>` | `<div role="complementary">` |
| Section with label | `<section aria-label>` | `<div role="region" aria-label>` |
| Headings | `<h1>` – `<h6>` | `<div role="heading" aria-level>` |
| Lists | `<ul>` / `<ol>` + `<li>` | `<div role="list">` + `<div role="listitem">` |
| Required field | `required` attribute | `aria-required="true"` alone |
| Disabled control | `disabled` attribute | `aria-disabled="true"` alone |

**When reviewing code or writing new code, actively replace ARIA with native HTML equivalents wherever possible.** If you see `role="button"` on a `<div>`, refactor it to a `<button>`. If you see `aria-expanded` managing show/hide, consider if `<details>` would work.

## Core Principles

### 1. Semantic HTML Structure

Use native HTML elements to build well-structured documents:

- Use `<nav>`, `<main>`, `<header>`, `<footer>`, `<aside>`, `<section>` for landmarks
- Use `<h1>`–`<h6>` in logical order — never skip levels within a section
- Use `<ul>`/`<ol>` for lists, `<table>` for tabular data
- Use `<fieldset>` + `<legend>` to group related form controls
- Use `<label>` with `for` attribute (or wrapping) for every form input
- Use `<details>` + `<summary>` for disclosure widgets
- Use `<dialog>` for modals (with `.showModal()` for built-in focus trapping)
- Use `<output>` for computation results and live status
- Use `<a href>` for navigation, `<button>` for actions — never the reverse

### 2. ARIA Usage Rules

When ARIA is necessary, follow these rules strictly:

1. **No ARIA is better than bad ARIA** — incorrect ARIA is worse than no ARIA
2. **Never change native semantics** unless absolutely necessary (don't put `role="heading"` on a `<button>`)
3. **All interactive ARIA controls must be keyboard operable**
4. **Never use `role="presentation"` or `aria-hidden="true"` on focusable elements**
5. **All interactive elements must have accessible names** — via label, `aria-label`, or `aria-labelledby`

Common ARIA patterns:
- `aria-expanded` + `aria-controls` for disclosure widgets
- `aria-haspopup` for menus and popups
- `aria-live` regions for dynamic content updates (`polite` for non-urgent, `assertive` for critical)
- `aria-describedby` for supplemental descriptions (error messages, help text)
- `aria-current="page"` for current navigation item

See `references/aria-patterns.md` for complete pattern library.

### 3. Keyboard Navigation

Every interactive element must be operable with keyboard alone:

**Tab order:**
- Follow visual/logical reading order
- Use `tabindex="0"` to add elements to tab order (rare — prefer native interactive elements)
- Use `tabindex="-1"` for programmatic focus (e.g., modal containers, error summaries)
- **Never use `tabindex` > 0** — it breaks natural tab order

**Key bindings by pattern:**
| Pattern | Keys |
|---|---|
| Buttons | Enter, Space |
| Links | Enter |
| Tabs | Arrow Left/Right, Home, End |
| Menus | Arrow Up/Down, Enter, Escape |
| Dialogs | Escape to close, Tab trapped inside |
| Combobox | Arrow Down to open, Escape to close |
| Tree view | Arrow Up/Down/Left/Right |

**Focus management rules:**
- When opening a modal, move focus to the first focusable element or the dialog itself
- When closing a modal, return focus to the trigger element
- After deleting an item, move focus to the next item or container
- Skip links: provide "Skip to main content" as the first focusable element
- Focus must be visible — never `outline: none` without a replacement

### 4. Forms & Validation

**Labels and instructions:**
- Every input MUST have a visible `<label>` (or `aria-label` for icon-only inputs)
- Group related inputs with `<fieldset>` + `<legend>`
- Provide instructions before the form, not only after errors
- Mark required fields with both visual indicator and `aria-required="true"` or `required`

**Error handling:**
- Use `aria-invalid="true"` on the invalid field
- Associate error messages with `aria-describedby`
- Provide an error summary at the top of the form, linked with `aria-describedby` or focused on submission
- Error messages must identify the field and describe how to fix the error
- Don't rely solely on color to indicate errors — use text and/or icons

**Input types:**
- Use appropriate `type` attributes: `email`, `tel`, `url`, `number`, `date`
- Use `autocomplete` attributes for common fields (name, email, address)
- Provide `inputmode` for mobile keyboard optimization

### 5. Color & Contrast

**WCAG 2.2 AA minimums:**
- **Normal text** (< 18pt / < 14pt bold): 4.5:1 contrast ratio
- **Large text** (>= 18pt / >= 14pt bold): 3:1 contrast ratio
- **UI components and graphical objects**: 3:1 against adjacent colors
- **Focus indicators**: 3:1 contrast, at least 2px wide

**Rules:**
- Never convey information by color alone — always pair with text, pattern, or icon
- Test with both light and dark color schemes
- Ensure links are distinguishable from surrounding text (underline or 3:1 contrast + non-color indicator on hover/focus)
- Placeholder text is NOT a label — it disappears and often has insufficient contrast

### 6. Images & Media

**Images:**
- Informative images: descriptive `alt` text conveying the purpose/content
- Decorative images: `alt=""` (empty) or use CSS background-image
- Complex images (charts, diagrams): brief `alt` + long description via `aria-describedby` or adjacent text
- Images of text: avoid when possible; if used, `alt` must contain the full text
- SVG icons: `aria-hidden="true"` when decorative; `role="img"` + `aria-label` when meaningful

**Media:**
- Video: captions (synchronized) + audio description when visual content isn't described in dialogue
- Audio: transcript provided
- Auto-playing media: must be pausable; prefer no auto-play

### 7. Motion & Animation

- Respect `prefers-reduced-motion: reduce` — disable or simplify all non-essential animations
- No content that flashes more than 3 times per second
- Provide pause/stop controls for any auto-moving content
- Carousels/sliders must be pausable and keyboard navigable

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

### 8. Document Structure

- One `<main>` per page
- Unique, descriptive `<title>` per page
- Language declared with `lang` attribute on `<html>`
- Landmark regions: `<header>`, `<nav>`, `<main>`, `<footer>`, `<aside>`
- If multiple `<nav>` or `<aside>`, label each with `aria-label`
- Heading hierarchy must be logical (h1 > h2 > h3) — don't skip levels

### 9. Responsive Accessibility

- Content must be usable at 400% zoom without loss of content or functionality
- Touch targets: minimum 24x24 CSS pixels (WCAG 2.2), prefer 44x44
- Content must reflow in a single column at 320px viewport width
- Don't disable pinch-to-zoom (`user-scalable=no` or `maximum-scale=1`)

## Code Review Rules

When reviewing code, flag these common violations:

1. **Missing labels** — inputs without associated `<label>`, `aria-label`, or `aria-labelledby`
2. **Click handlers on non-interactive elements** — `onClick` on `<div>` or `<span>` without `role`, `tabindex`, and keyboard handling
3. **Missing alt text** — `<img>` without `alt` attribute
4. **Incorrect heading levels** — skipped heading levels or multiple `<h1>`
5. **Color-only indicators** — status, errors, or states conveyed only by color
6. **Missing focus styles** — `outline: none` or `:focus { outline: 0 }` without replacement
7. **Inaccessible custom controls** — dropdowns, tabs, modals without ARIA and keyboard support
8. **Auto-playing media** without pause control
9. **Missing landmark regions** — no `<main>`, no `<nav>`
10. **Form errors without programmatic association** — error text not linked via `aria-describedby`

## Browser Verification with MCP Tools

When verifying accessibility in the browser, use chrome-devtools MCP tools:

### Inspect the Accessibility Tree
Use `take_snapshot` to get the a11y tree and verify:
- Elements have correct roles (button, link, heading, etc.)
- Interactive elements have accessible names
- ARIA states are properly set (expanded, selected, checked, etc.)
- Landmark regions are present and labeled

### Test Keyboard Navigation
Use `press_key` to verify keyboard-only operation:
- **Tab** / **Shift+Tab**: moves focus through interactive elements in logical order
- **Enter** / **Space**: activates buttons and links
- **Escape**: closes modals, menus, popups
- **Arrow keys**: navigates within composite widgets (tabs, menus, trees)
- Verify focus is visible on every interactive element
- Verify focus is trapped inside open modals

### Run Automated Checks
Use `evaluate_script` to run axe-core in the page:
```javascript
// If axe-core is loaded on the page
async () => {
  const results = await axe.run();
  return {
    violations: results.violations.map(v => ({
      id: v.id,
      impact: v.impact,
      description: v.description,
      nodes: v.nodes.length
    }))
  };
}
```

### Test Color Schemes
Use `emulate` to verify accessibility across preferences:
- `colorScheme: "dark"` — verify contrast ratios hold in dark mode
- `colorScheme: "light"` — verify light mode
- Verify `prefers-reduced-motion` behavior by checking that animation-dependent content remains accessible

## Testing

Defer to the `testing` skill for test tooling specifics (Vitest, Playwright, axe-core integration). This skill owns the **what** to test; `testing` owns the **how**.

**What to test for accessibility:**
- All interactive elements are keyboard reachable and operable
- ARIA attributes are correctly set for each state
- Error messages are programmatically associated with inputs
- Focus management works correctly (modals, deletions, route changes)
- axe-core reports zero violations at AA level
- Content is usable at 200% and 400% zoom
- Color scheme changes don't break contrast

See `references/wcag-22-aa-checklist.md` for the complete checklist and `references/testing-procedures.md` for verification procedures.

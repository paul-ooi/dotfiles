# WCAG 2.2 AA Checklist

Quick-reference checklist organized by principle. Each item maps to a WCAG 2.2 success criterion. Check quarterly if there are newer released versions of the WCAG spec and update this checklist accordingly.

## Perceivable

### Text Alternatives (1.1)
- [ ] All `<img>` elements have appropriate `alt` text (or `alt=""` for decorative)
- [ ] Complex images have extended descriptions
- [ ] SVG icons used as content have `role="img"` + `aria-label`
- [ ] Decorative SVGs have `aria-hidden="true"`
- [ ] `<input type="image">` has descriptive `alt`

### Time-Based Media (1.2)
- [ ] Pre-recorded video has synchronized captions
- [ ] Pre-recorded audio has a transcript
- [ ] Pre-recorded video has audio description (when visual info not in dialogue)
- [ ] Live video has captions

### Adaptable (1.3)
- [ ] Information and structure conveyed through presentation are available programmatically
- [ ] Use native HTML elements for structure: `<h1>`–`<h6>`, `<ul>`, `<ol>`, `<table>`, `<fieldset>`
- [ ] Reading order in DOM matches visual order
- [ ] Instructions don't rely solely on shape, size, or position
- [ ] Content doesn't restrict to a single orientation (portrait/landscape)
- [ ] Input purpose is identified via `autocomplete` for common fields (2.2 — 1.3.5)

### Distinguishable (1.4)
- [ ] Color is not the sole means of conveying information
- [ ] Normal text contrast >= 4.5:1
- [ ] Large text contrast >= 3:1
- [ ] Text can be resized up to 200% without loss of content
- [ ] Images of text avoided (or have equivalent alt)
- [ ] Content reflows at 320px width (no horizontal scrolling)
- [ ] UI component contrast >= 3:1 against adjacent colors
- [ ] Text spacing can be overridden (line height, letter spacing, word spacing, paragraph spacing) without loss
- [ ] Hover/focus content is dismissible, hoverable, and persistent (1.4.13)

## Operable

### Keyboard Accessible (2.1)
- [ ] All functionality available via keyboard
- [ ] No keyboard traps (except intentional, like modals with documented Escape)
- [ ] Keyboard shortcuts can be turned off or remapped (if single-character shortcuts exist)

### Enough Time (2.2)
- [ ] Time limits are adjustable (extend, turn off)
- [ ] Auto-updating content can be paused, stopped, or hidden
- [ ] No timeouts that cause data loss (or user is warned)

### Seizures and Physical Reactions (2.3)
- [ ] No content flashes more than 3 times per second

### Navigable (2.4)
- [ ] Skip navigation link provided
- [ ] Pages have descriptive `<title>`
- [ ] Focus order matches logical reading order
- [ ] Link purpose is clear from link text (or link + context)
- [ ] Multiple ways to find pages (nav, search, sitemap)
- [ ] Headings and labels are descriptive
- [ ] Focus is visible on all interactive elements
- [ ] Focus indicator has >= 3:1 contrast and >= 2px (2.2 — 2.4.13 Focus Appearance)

### Input Modalities (2.5)
- [ ] Pointer gestures have single-pointer alternatives
- [ ] Pointer actions can be cancelled (up-event or undo)
- [ ] Visible labels match accessible names (label-in-name)
- [ ] Functionality via motion (shake, tilt) has UI alternative
- [ ] Touch targets are at least 24x24 CSS pixels (2.2 — 2.5.8 Target Size Minimum)

## Understandable

### Readable (3.1)
- [ ] Page language identified with `lang` attribute on `<html>`
- [ ] Language changes within page marked with `lang` attribute

### Predictable (3.2)
- [ ] Focus changes don't cause unexpected context changes
- [ ] Input changes don't cause unexpected context changes
- [ ] Navigation is consistent across pages
- [ ] Components identified consistently across pages
- [ ] Help mechanisms are in consistent locations (2.2 — 3.2.6 Consistent Help)

### Input Assistance (3.3)
- [ ] Errors are identified and described in text
- [ ] Labels and instructions provided for inputs
- [ ] Error suggestions provided when known
- [ ] Submissions are reversible, verified, or confirmed (for legal/financial)
- [ ] Redundant entry: info previously entered is auto-populated or selectable (2.2 — 3.3.7)
- [ ] Accessible authentication: no cognitive function test for login (2.2 — 3.3.8)

## Robust

### Compatible (4.1)
- [ ] HTML validates (no duplicate IDs, proper nesting)
- [ ] All custom components have appropriate name, role, value
- [ ] Status messages use `aria-live` or appropriate roles (status, alert, log, timer, marquee)

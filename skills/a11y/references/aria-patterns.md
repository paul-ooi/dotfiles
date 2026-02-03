# ARIA Patterns Reference

## The Cardinal Rule: Prefer Native HTML

**ARIA is a last resort.** Before using any ARIA pattern below, verify that no native HTML element or combination of elements can achieve the same result. Native HTML gives you keyboard behavior, focus management, form participation, and screen reader semantics for free.

| Instead of ARIA... | Use native HTML |
|---|---|
| `role="button"` + `tabindex="0"` + keydown handler | `<button>` |
| `role="link"` | `<a href="...">` |
| `role="checkbox"` + `aria-checked` | `<input type="checkbox">` |
| `role="radio"` + `aria-checked` | `<input type="radio">` |
| `role="textbox"` | `<input>` or `<textarea>` |
| `role="heading"` + `aria-level` | `<h1>`–`<h6>` |
| `role="list"` + `role="listitem"` | `<ul>` + `<li>` |
| `role="navigation"` | `<nav>` |
| `role="main"` | `<main>` |
| `role="banner"` | `<header>` (page-level) |
| `role="contentinfo"` | `<footer>` (page-level) |
| `role="complementary"` | `<aside>` |
| `role="region"` + `aria-label` | `<section aria-label="...">` |
| `role="figure"` + `aria-label` | `<figure>` + `<figcaption>` |
| `role="table"` + row/cell roles | `<table>` + `<thead>`, `<tbody>`, `<tr>`, `<th>`, `<td>` |
| `role="group"` + `aria-label` | `<fieldset>` + `<legend>` |
| `role="progressbar"` | `<progress>` |
| `role="meter"` | `<meter>` |
| `role="separator"` (non-interactive) | `<hr>` |
| `role="status"` | `<output>` (for form-related status) |
| `aria-required="true"` | `required` attribute (HTML5) |

**Only use ARIA when there is no native HTML element that provides the needed semantics or behavior.** The patterns below are for those cases.

---

## Disclosure (Show/Hide)

**Native-first:** Use `<details>` + `<summary>` when the pattern fits (simple show/hide of content). Only use the ARIA pattern below when you need custom behavior that `<details>` cannot provide (e.g., animation, external trigger, or the toggle is not adjacent to the content).

```html
<!-- Native HTML — prefer this -->
<details>
  <summary>More information</summary>
  <p>Additional content here.</p>
</details>

<!-- ARIA pattern — only when <details> is insufficient -->
<button aria-expanded="false" aria-controls="content-1">
  More information
</button>
<div id="content-1" hidden>
  <p>Additional content here.</p>
</div>
```

**Keyboard:** Enter/Space toggles. Manage `aria-expanded` and `hidden` attribute.

---

## Modal Dialog

**No native equivalent exists that provides full modal behavior with focus trapping.** Use ARIA.

```html
<div role="dialog" aria-modal="true" aria-labelledby="dialog-title">
  <h2 id="dialog-title">Confirm deletion</h2>
  <p>Are you sure you want to delete this item?</p>
  <button>Cancel</button>
  <button>Delete</button>
</div>
```

**Note:** The `<dialog>` HTML element is now well-supported. Prefer `<dialog>` with its `showModal()` method — it provides built-in focus trapping and Escape to close:

```html
<!-- Preferred: native <dialog> -->
<dialog id="confirm-dialog" aria-labelledby="dialog-title">
  <h2 id="dialog-title">Confirm deletion</h2>
  <p>Are you sure you want to delete this item?</p>
  <button>Cancel</button>
  <button>Delete</button>
</dialog>
```

**Requirements:**
- Focus moves to dialog (or first focusable element) on open
- Focus is trapped inside while open
- Escape closes the dialog
- Focus returns to trigger element on close
- Background content is inert (`inert` attribute on siblings, or managed by `<dialog>.showModal()`)

---

## Tabs

**No native tab element exists.** Use ARIA.

```html
<div role="tablist" aria-label="Account settings">
  <button role="tab" id="tab-1" aria-selected="true" aria-controls="panel-1">
    Profile
  </button>
  <button role="tab" id="tab-2" aria-selected="false" aria-controls="panel-2" tabindex="-1">
    Security
  </button>
</div>
<div role="tabpanel" id="panel-1" aria-labelledby="tab-1">
  <!-- Profile content -->
</div>
<div role="tabpanel" id="panel-2" aria-labelledby="tab-2" hidden>
  <!-- Security content -->
</div>
```

**Keyboard:**
- Arrow Left/Right moves between tabs
- Home/End moves to first/last tab
- Only the active tab is in the tab order (`tabindex="0"`); others are `tabindex="-1"`
- Tab key moves focus into the panel content

---

## Menu / Menu Button

**No native equivalent for a custom dropdown menu with actions.** Use ARIA. (If you only need a list of links, use a disclosure pattern with a `<ul>` of `<a>` elements instead.)

```html
<button aria-haspopup="true" aria-expanded="false" aria-controls="actions-menu">
  Actions
</button>
<ul role="menu" id="actions-menu" hidden>
  <li role="menuitem">Edit</li>
  <li role="menuitem">Duplicate</li>
  <li role="separator"></li>
  <li role="menuitem">Delete</li>
</ul>
```

**Keyboard:**
- Enter/Space or Arrow Down opens menu, focuses first item
- Arrow Up/Down navigates items
- Escape closes menu, returns focus to button
- Type-ahead: typing a letter focuses matching item

---

## Combobox (Autocomplete)

**No native autocomplete element exists with suggestion list.** Use ARIA. (For simple selection, prefer `<select>` or `<datalist>` when they meet requirements.)

```html
<!-- Prefer <datalist> for simple cases -->
<input type="text" list="suggestions">
<datalist id="suggestions">
  <option value="Apple">
  <option value="Banana">
</datalist>

<!-- ARIA combobox for complex cases (custom rendering, async, etc.) -->
<label for="search">Search</label>
<input id="search" role="combobox" aria-expanded="false"
       aria-autocomplete="list" aria-controls="search-listbox">
<ul id="search-listbox" role="listbox" hidden>
  <li role="option" id="opt-1">Option 1</li>
  <li role="option" id="opt-2">Option 2</li>
</ul>
```

**Keyboard:**
- Arrow Down opens list / moves through options
- Enter selects focused option
- Escape closes list
- `aria-activedescendant` tracks the focused option

---

## Accordion

**Native-first:** Use multiple `<details>` + `<summary>` elements. Only use ARIA when you need single-open behavior or custom interaction.

```html
<!-- Native HTML — prefer this for most accordions -->
<details>
  <summary>Section 1</summary>
  <p>Content for section 1.</p>
</details>
<details>
  <summary>Section 2</summary>
  <p>Content for section 2.</p>
</details>

<!-- ARIA pattern — for single-open or custom behavior -->
<h3>
  <button aria-expanded="true" aria-controls="section-1">Section 1</button>
</h3>
<div id="section-1" role="region" aria-labelledby="section-1-heading">
  <p>Content for section 1.</p>
</div>
```

---

## Tooltip

**Native-first:** For simple text tooltips, use the `title` attribute. For richer tooltips, use ARIA.

```html
<!-- Simple — title attribute (limited styling, but accessible) -->
<button title="Save your document">Save</button>

<!-- ARIA tooltip — when you need styled, persistent tooltips -->
<button aria-describedby="tooltip-1">Save</button>
<div id="tooltip-1" role="tooltip" hidden>
  Save your document (Ctrl+S)
</div>
```

**Requirements:**
- Appears on hover and focus
- Dismissible with Escape
- Does not contain interactive content (use a popover/dialog instead)

---

## Alert / Status Messages

**Native-first:** Use `<output>` for form computation results. Use ARIA live regions for dynamic status that must be announced.

```html
<!-- Form result — prefer <output> -->
<output aria-live="polite">3 results found</output>

<!-- Alert — important, time-sensitive -->
<div role="alert">Your session will expire in 2 minutes.</div>

<!-- Status — non-urgent update -->
<div role="status">File uploaded successfully.</div>

<!-- Log — sequential updates -->
<div role="log" aria-live="polite">
  <p>Connected to server.</p>
  <p>Loading data...</p>
</div>
```

`role="alert"` is implicitly `aria-live="assertive"`. `role="status"` is implicitly `aria-live="polite"`.

---

## Tree View

**No native equivalent.** Use ARIA.

```html
<ul role="tree" aria-label="File explorer">
  <li role="treeitem" aria-expanded="true">
    src/
    <ul role="group">
      <li role="treeitem">index.js</li>
      <li role="treeitem">App.js</li>
    </ul>
  </li>
</ul>
```

**Keyboard:**
- Arrow Up/Down moves between visible items
- Arrow Right expands / moves to first child
- Arrow Left collapses / moves to parent
- Home/End moves to first/last visible item

---

## Switch (Toggle)

**Native-first:** Use `<input type="checkbox">` with appropriate styling. Only use `role="switch"` when the UI clearly represents an on/off toggle (not a checkbox).

```html
<!-- Prefer styled checkbox for most cases -->
<label>
  <input type="checkbox"> Enable notifications
</label>

<!-- role="switch" — only for explicit on/off toggle UI -->
<button role="switch" aria-checked="false">
  Dark mode
</button>
```

---

## General ARIA Properties Quick Reference

| Property | Purpose |
|---|---|
| `aria-label` | Names an element when no visible text label exists |
| `aria-labelledby` | Points to element(s) that label this element |
| `aria-describedby` | Points to element(s) providing additional description |
| `aria-expanded` | Indicates whether a collapsible section is open |
| `aria-controls` | Identifies the element controlled by this element |
| `aria-haspopup` | Indicates a popup (menu, dialog, etc.) will appear |
| `aria-live` | Defines a live region (`polite`, `assertive`, `off`) |
| `aria-current` | Identifies the current item (`page`, `step`, `date`, etc.) |
| `aria-invalid` | Marks a form field as having an invalid value |
| `aria-required` | Marks a field as required (prefer HTML `required` attribute) |
| `aria-hidden` | Hides element from assistive technology (never on focusable elements) |
| `aria-disabled` | Marks as disabled (prefer HTML `disabled` attribute on native controls) |
| `aria-busy` | Indicates a region is being updated |
| `aria-activedescendant` | Identifies the focused child in composite widgets |

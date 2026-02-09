---
name: component-design
description: Designs accessible, reusable component architectures for web UI libraries and design systems. Covers component tiers, API design, compound components, headless patterns, composition, and slots. Use when building components, design systems, defining props interfaces, or structuring compound and headless components.
compatibility: Designed for Claude Code.
---

# Component Design — Accessible, Reusable Component Architecture

This skill provides patterns for building component libraries and design systems. Framework-agnostic principles with practical examples. Accessibility is built in, not bolted on.

## Component Tiers

Organize components into three tiers based on scope and reusability:

### Tier 1: Primitives
Low-level building blocks. High reuse, minimal opinions, maximum flexibility.

- Button, Input, Select, Checkbox, Radio
- Text, Heading, Link, Icon
- Stack, Grid, Cluster (layout primitives)
- Dialog, Tooltip, Popover (overlay primitives)

**Rules for primitives:**
- Wrap a single native HTML element (or small, fixed group)
- Forward all relevant HTML attributes to the root element
- Accept `className` for style overrides
- Accessibility is built in — correct roles, keyboard behavior, ARIA by default
- No business logic, no data fetching
- Use native HTML elements as the foundation (see `a11y` skill for native-first principle)

### Tier 2: Composites
Combine primitives into common UI patterns. Moderate reuse.

- SearchField (Input + Button + suggestions)
- FormField (Label + Input + error message)
- Card (surface + layout + content slots)
- Pagination (buttons + page indicators)

**Rules for composites:**
- Compose primitives — don't reimplement their internals
- Expose a focused API (not every primitive prop surfaces)
- Maintain accessibility contracts established by primitives

### Tier 3: Features
Domain-specific components. Low reuse, high business context.

- UserProfileCard, InvoiceTable, CheckoutForm
- NavigationSidebar, DashboardWidget

**Rules for features:**
- Use composites and primitives internally
- May contain business logic and data fetching
- Not expected to be reusable across projects

## Component API Design

### Props Rules

1. **Minimal surface area** — fewer props means easier to learn, use, and maintain
2. **Forward HTML attributes** — don't create custom props for things HTML already handles (`id`, `className`, `aria-*`, `data-*`)
3. **Boolean props default to `false`** — `<Button disabled>` not `<Button enabled={false}>`
4. **Prefer enums over multiple booleans** — `variant="primary"` not `isPrimary={true}`
5. **Children/slots for content** — don't pass JSX as props when children/slots work
6. **Callback naming** — `onAction` for events (`onClick`, `onChange`, `onDismiss`)
7. **No `className` override of internal structure** — expose design-sanctioned variants instead

```typescript
// Good API
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  loading?: boolean;
  onClick?: (event: MouseEvent) => void;
  children: ReactNode; // or framework equivalent
  // Plus forwarded HTML button attributes
}

// Bad API
interface ButtonProps {
  isPrimary?: boolean;
  isSecondary?: boolean;
  isGhost?: boolean;
  isSmall?: boolean;
  isLarge?: boolean;
  text?: string;           // Use children instead
  iconLeft?: ReactNode;    // Okay — positional slots
  onPress?: () => void;    // Non-standard name; use onClick
  style?: CSSProperties;   // Breaks encapsulation
}
```

### Forwarding Attributes and Refs

Primitives must forward attributes and refs to the underlying HTML element:

```typescript
// Conceptual — framework-agnostic pattern
// The component accepts all valid HTML button attributes
// and forwards them, plus a ref, to the native <button>
function Button({ variant, size, children, ...htmlProps }) {
  return (
    <button
      className={getClassNames(variant, size)}
      {...htmlProps}
    >
      {children}
    </button>
  );
}
```

This ensures consumers can add `aria-*`, `data-*`, event handlers, or any other HTML attribute without the component having to explicitly support each one.

## Composition Patterns

### Compound Components

Use compound components when a widget has multiple related sub-parts with shared state:

```jsx
// Usage — readable, flexible ordering
<Tabs defaultValue="profile">
  <Tabs.List>
    <Tabs.Trigger value="profile">Profile</Tabs.Trigger>
    <Tabs.Trigger value="settings">Settings</Tabs.Trigger>
  </Tabs.List>
  <Tabs.Panel value="profile">Profile content</Tabs.Panel>
  <Tabs.Panel value="settings">Settings content</Tabs.Panel>
</Tabs>
```

**When to use:** Tabs, Accordion, Select, Menu, Dialog, RadioGroup — anywhere sub-parts need to communicate through shared context.

**Rules:**
- Parent provides context (state, callbacks)
- Children consume context — no prop drilling
- Order of children is flexible (consumer controls layout)
- Each sub-part manages its own accessibility attributes

### Headless Components

Separate behavior from presentation when multiple visual implementations need the same interaction logic:

```javascript
// Headless hook — manages state and ARIA, no rendering
function useToggle({ defaultOpen = false } = {}) {
  const [isOpen, setIsOpen] = useState(defaultOpen);
  return {
    isOpen,
    toggle: () => setIsOpen(v => !v),
    triggerProps: {
      'aria-expanded': isOpen,
      onClick: () => setIsOpen(v => !v),
    },
    contentProps: {
      hidden: !isOpen,
    },
  };
}
```

**When to use:** When the same interaction pattern (toggle, combobox, sortable list) appears with different visual designs across the app.

### Composition Over Configuration

**Prefer:**
```jsx
<Card>
  <Card.Header>
    <Heading level={3}>Title</Heading>
    <Badge>New</Badge>
  </Card.Header>
  <Card.Body>Content here</Card.Body>
  <Card.Footer>
    <Button>Action</Button>
  </Card.Footer>
</Card>
```

**Over:**
```jsx
<Card
  title="Title"
  badge="New"
  body="Content here"
  footer={<Button>Action</Button>}
/>
```

Configuration props create rigid APIs that can't accommodate unforeseen layouts. Composition (children/slots) lets consumers control structure.

## Accessibility Built In

Components must be accessible by default — consumers shouldn't need to add ARIA attributes for standard use cases.

**Primitive responsibilities:**
- Correct HTML element (`<button>`, `<a>`, `<input>`, `<dialog>`, etc.) — native first, ARIA only as last resort (defer to `a11y` skill)
- Keyboard behavior built in (Enter/Space for buttons, Escape for dialogs, Arrow keys for composite widgets)
- Focus management (trap in dialogs, return on close)
- ARIA attributes managed internally for state (`aria-expanded`, `aria-selected`, `aria-checked`)

**Consumer responsibilities:**
- Provide text content (labels, headings)
- Provide `aria-label` when there's no visible text (icon-only buttons)
- Follow usage patterns documented in the component API

```jsx
// The Dialog component handles focus trap, Escape, aria-modal, role
// The consumer only needs to provide content
<Dialog open={isOpen} onDismiss={close}>
  <Dialog.Title>Confirm Action</Dialog.Title>
  <Dialog.Body>Are you sure?</Dialog.Body>
  <Dialog.Actions>
    <Button onClick={close}>Cancel</Button>
    <Button variant="primary" onClick={confirm}>Confirm</Button>
  </Dialog.Actions>
</Dialog>
```

## Styling Integration

Use CSS Modules for component styles. Defer to `css-styling` skill for detailed patterns.

**Key integration points:**
- One `.module.css` file per component
- Variants implemented as CSS classes, toggled by props
- All colors, spacing, and sizes via CSS custom property tokens
- Components inherit dark/light mode from global tokens automatically
- Accept `className` prop for consumer overrides (applied to root element only)

```css
/* Dialog.module.css */
.overlay {
  position: fixed;
  inset: 0;
  background: var(--color-overlay);
  display: grid;
  place-items: center;
}

.dialog {
  background: var(--color-surface-raised);
  border-radius: var(--radius-lg);
  padding: var(--space-6);
  max-inline-size: min(500px, 90vw);
  box-shadow: var(--shadow-xl);
}
```

## Controlled vs. Uncontrolled

Support both patterns when a component manages state:

```typescript
interface SelectProps {
  // Controlled
  value?: string;
  onChange?: (value: string) => void;
  // Uncontrolled
  defaultValue?: string;
}
```

**Rules:**
- `value` + `onChange` = controlled (consumer owns the state)
- `defaultValue` = uncontrolled (component owns the state, consumer sets initial)
- Providing both `value` and `defaultValue` is an error — warn in development
- Internal state should sync with `value` prop when controlled

## Event Naming

- Mirror native HTML event names: `onClick`, `onChange`, `onFocus`, `onBlur`
- For custom events: `on` + past-tense action: `onDismiss`, `onSelect`, `onExpand`
- Pass the original event when wrapping native events — don't swallow it
- For custom events, provide relevant data: `onSelect(value)`, `onPageChange(page)`

## File Structure Per Component

```
Button/
  Button.tsx           # Component implementation
  Button.module.css    # Styles
  Button.test.tsx      # Tests (defer to testing skill)
  index.ts             # Re-export: export { Button } from './Button'
```

For compound components:
```
Tabs/
  Tabs.tsx             # Parent + context provider
  TabsList.tsx         # Sub-component
  TabsTrigger.tsx      # Sub-component
  TabsPanel.tsx        # Sub-component
  Tabs.module.css      # All tab styles
  Tabs.test.tsx        # Tests
  index.ts             # Export compound: Tabs.List, Tabs.Trigger, Tabs.Panel
```

## Anti-Patterns

### God Components
Components that do everything — fetch data, manage complex state, render large templates, handle routing. Split into feature (orchestration) and presentational (rendering) layers.

### Prop Drilling
Passing props through 3+ levels of components that don't use them. Use context, composition, or state management instead.

### Premature Abstraction
Creating a `GenericList` component before you have 3 concrete examples of lists. Write the concrete components first — extract when patterns emerge (rule of three from `clean-code` skill).

### Boolean Prop Explosion
```jsx
// Anti-pattern: which combinations are valid?
<Button primary secondary outline large small loading disabled />

// Better: constrained enums
<Button variant="primary" size="lg" loading disabled />
```

### CSS Leaking
Targeting component internals from outside:
```css
/* Anti-pattern: parent reaching into child internals */
.page .button .icon { color: red; }

/* Better: component exposes a prop or CSS variable */
<Button iconColor="error" />
```

### Reinventing Native HTML
Building a custom `<Select>` from `<div>` elements when the native `<select>` would work. Always evaluate if native HTML elements meet the requirements before building custom widgets (defer to `a11y` skill for native-first approach).

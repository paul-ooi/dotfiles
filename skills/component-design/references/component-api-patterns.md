# Component API Patterns

## Contents
- Prop Spread Pattern
- Polymorphic "as" Pattern
- Slot Pattern
- Render Prop / Function-as-Child
- Controlled/Uncontrolled Dual Pattern
- Context Pattern for Compound Components
- Size Variants Pattern
- Loading State Pattern
- Error State Pattern for Form Fields
- Visually Hidden Utility

## Prop Spread Pattern

Forward unknown props to the root HTML element so consumers can add `aria-*`, `data-*`, event handlers, etc.:

```typescript
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
}

function Button({ variant = 'secondary', size = 'md', loading, children, className, ...props }: ButtonProps) {
  return (
    <button
      className={clsx(styles.root, styles[variant], styles[size], className)}
      disabled={loading || props.disabled}
      {...props}
    >
      {loading && <Spinner aria-hidden="true" />}
      {children}
    </button>
  );
}
```

**Framework-agnostic principle:** Whatever your framework, the pattern is the same — accept known props, spread the rest to the DOM element.

## Polymorphic "as" Pattern

Let consumers change the rendered element while keeping component styles and behavior:

```typescript
// Conceptual pattern
function Button({ as: Element = 'button', ...props }) {
  return <Element className={styles.root} {...props} />;
}

// Usage
<Button>Click me</Button>                    // renders <button>
<Button as="a" href="/page">Go</Button>     // renders <a>
```

**Use sparingly.** Most components should render a single, correct HTML element. The `as` pattern is appropriate for:
- Text components that render different heading levels (`as="h2"`)
- Navigation items that can be links or buttons
- Layout primitives that can be different sectioning elements

## Slot Pattern

For components that need to accept content in specific positions:

```jsx
// Named slots via props — for fixed positions
function Card({ header, footer, children }) {
  return (
    <div className={styles.root}>
      {header && <div className={styles.header}>{header}</div>}
      <div className={styles.body}>{children}</div>
      {footer && <div className={styles.footer}>{footer}</div>}
    </div>
  );
}

// Usage
<Card
  header={<h3>Title</h3>}
  footer={<Button>Save</Button>}
>
  <p>Card content</p>
</Card>
```

**Prefer compound components over slots** when the content structure is complex or variable.

## Render Prop / Function-as-Child

Expose internal state to the consumer for maximum rendering flexibility:

```jsx
function Disclosure({ children, defaultOpen = false }) {
  const [isOpen, setIsOpen] = useState(defaultOpen);
  return children({
    isOpen,
    toggle: () => setIsOpen(v => !v),
    triggerProps: { 'aria-expanded': isOpen, onClick: () => setIsOpen(v => !v) },
    contentProps: { hidden: !isOpen },
  });
}

// Usage
<Disclosure>
  {({ triggerProps, contentProps }) => (
    <>
      <button {...triggerProps}>Toggle</button>
      <div {...contentProps}>Hidden content</div>
    </>
  )}
</Disclosure>
```

**Use when:** consumers need access to internal state to make rendering decisions. Headless hooks are often a cleaner alternative.

## Controlled/Uncontrolled Dual Pattern

Implementation pattern for supporting both modes:

```typescript
function useControllableState<T>({
  value: controlledValue,
  defaultValue,
  onChange,
}: {
  value?: T;
  defaultValue: T;
  onChange?: (value: T) => void;
}) {
  const [internalValue, setInternalValue] = useState(defaultValue);
  const isControlled = controlledValue !== undefined;
  const value = isControlled ? controlledValue : internalValue;

  const setValue = useCallback((next: T) => {
    if (!isControlled) {
      setInternalValue(next);
    }
    onChange?.(next);
  }, [isControlled, onChange]);

  return [value, setValue] as const;
}
```

## Context Pattern for Compound Components

```typescript
// 1. Create context
const TabsContext = createContext<TabsContextValue | null>(null);

function useTabsContext() {
  const context = useContext(TabsContext);
  if (!context) {
    throw new Error('Tabs compound components must be used within <Tabs>');
  }
  return context;
}

// 2. Provider in parent
function Tabs({ defaultValue, children }) {
  const [activeTab, setActiveTab] = useState(defaultValue);
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      {children}
    </TabsContext.Provider>
  );
}

// 3. Consumers
function TabsTrigger({ value, children }) {
  const { activeTab, setActiveTab } = useTabsContext();
  const isSelected = activeTab === value;
  return (
    <button
      role="tab"
      aria-selected={isSelected}
      tabIndex={isSelected ? 0 : -1}
      onClick={() => setActiveTab(value)}
    >
      {children}
    </button>
  );
}

// 4. Attach sub-components
Tabs.List = TabsList;
Tabs.Trigger = TabsTrigger;
Tabs.Panel = TabsPanel;
```

## Size Variants Pattern

Consistent sizing across components:

```css
/* sizes.module.css — shared */
.sm { --component-height: 2rem; --component-font: var(--text-sm); --component-px: var(--space-2); }
.md { --component-height: 2.5rem; --component-font: var(--text-base); --component-px: var(--space-4); }
.lg { --component-height: 3rem; --component-font: var(--text-lg); --component-px: var(--space-6); }
```

```css
/* Button.module.css */
.root {
  height: var(--component-height);
  font-size: var(--component-font);
  padding-inline: var(--component-px);
}
```

## Loading State Pattern

```jsx
function Button({ loading, disabled, children, ...props }) {
  return (
    <button
      disabled={loading || disabled}
      aria-busy={loading || undefined}
      {...props}
    >
      {loading ? (
        <>
          <Spinner aria-hidden="true" />
          <span className="sr-only">Loading</span>
        </>
      ) : (
        children
      )}
    </button>
  );
}
```

**Rules:**
- `disabled` during loading prevents double-submission
- `aria-busy` signals loading state to screen readers
- Visible spinner is decorative (`aria-hidden`) — screen reader gets "Loading" text
- Preserve button dimensions during loading (don't shift layout)

## Error State Pattern for Form Fields

```jsx
function FormField({ label, error, required, children, id }) {
  const errorId = error ? `${id}-error` : undefined;
  const descId = errorId; // extend with help text ID if present

  return (
    <div>
      <label htmlFor={id}>
        {label}
        {required && <span aria-hidden="true"> *</span>}
      </label>
      {/* Clone child to inject aria attributes */}
      {cloneElement(children, {
        id,
        'aria-invalid': error ? true : undefined,
        'aria-describedby': descId,
        'aria-required': required,
      })}
      {error && (
        <p id={errorId} role="alert">
          {error}
        </p>
      )}
    </div>
  );
}
```

## Visually Hidden Utility

For content that should be accessible to screen readers but not visible:

```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}
```

**Use for:** skip links (until focused), icon-only button labels, form instructions that are visually implicit.

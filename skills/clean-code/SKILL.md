---
triggers:
  - refactor
  - clean code
  - SOLID
  - DRY
  - naming
  - code quality
  - code review
  - single responsibility
---

# Clean Code — SOLID / DRY / Code Quality

This skill provides principles for writing maintainable, readable code. Framework-agnostic — applies to any web project. These principles are foundational; all other skills assume clean code practices.

## SOLID Principles

### Single Responsibility Principle (SRP)

**A module/function/class should have one reason to change.**

Each unit of code should do one thing well. When a function handles both data fetching and DOM manipulation, changes to the API force changes to the rendering code.

**Signals of violation:**
- Function name includes "and" (`fetchAndRender`, `validateAndSave`)
- Changing one feature requires editing unrelated code
- Hard to name what a function does in a short phrase
- Function takes many unrelated parameters

**Web example — before:**
```javascript
function submitForm(formData) {
  // Validates
  if (!formData.email.includes('@')) throw new Error('Invalid email');
  // Transforms
  const payload = { ...formData, createdAt: Date.now() };
  // Sends
  return fetch('/api/users', { method: 'POST', body: JSON.stringify(payload) });
}
```

**After:**
```javascript
function validateUserForm(formData) {
  if (!formData.email.includes('@')) throw new Error('Invalid email');
}

function toUserPayload(formData) {
  return { ...formData, createdAt: Date.now() };
}

function createUser(payload) {
  return fetch('/api/users', { method: 'POST', body: JSON.stringify(payload) });
}
```

### Open/Closed Principle (OCP)

**Open for extension, closed for modification.**

Design systems where new behavior can be added without changing existing code. In web development, this often means:
- Configuration objects over switch statements
- Plugin/middleware patterns
- Component composition over modification

**Web example:**
```javascript
// Closed for modification: adding a new formatter doesn't change this function
const formatters = {
  currency: (value) => `$${value.toFixed(2)}`,
  percent: (value) => `${(value * 100).toFixed(1)}%`,
  date: (value) => new Intl.DateTimeFormat().format(value),
};

function formatValue(value, type) {
  const formatter = formatters[type];
  if (!formatter) return String(value);
  return formatter(value);
}

// Open for extension: add new formatters without touching formatValue
formatters.phone = (value) => value.replace(/(\d{3})(\d{3})(\d{4})/, '($1) $2-$3');
```

### Liskov Substitution Principle (LSP)

**Subtypes must be substitutable for their base types.**

In web development, this applies to:
- Components that implement a common interface (all buttons accept `onClick`, `disabled`)
- Functions that share a signature (all validators return the same shape)
- API response handlers that expect consistent shapes

**Key rule:** If your code checks `instanceof` or `typeof` to decide behavior, you're probably violating LSP.

### Interface Segregation Principle (ISP)

**Don't force consumers to depend on things they don't use.**

- Component props: don't pass an entire user object when only `name` is needed
- Function parameters: accept only what's needed, not a large options bag
- Module exports: export focused functions, not god objects

**Web example — before:**
```javascript
// Forces every consumer to know about the full User shape
function UserAvatar({ user }) {
  return <img src={user.avatarUrl} alt={user.name} />;
}
```

**After:**
```javascript
// Only requires what it uses
function UserAvatar({ src, name }) {
  return <img src={src} alt={name} />;
}
```

### Dependency Inversion Principle (DIP)

**Depend on abstractions, not concretions.**

High-level business logic should not import low-level implementation details directly. Use dependency injection or configuration:

```javascript
// High-level: depends on an abstract storage interface
function createTodoService(storage) {
  return {
    add: (todo) => storage.save('todos', todo),
    getAll: () => storage.load('todos'),
  };
}

// Low-level: concrete implementations
const localStorageAdapter = {
  save: (key, value) => localStorage.setItem(key, JSON.stringify(value)),
  load: (key) => JSON.parse(localStorage.getItem(key)),
};

const apiAdapter = {
  save: (key, value) => fetch(`/api/${key}`, { method: 'POST', body: JSON.stringify(value) }),
  load: (key) => fetch(`/api/${key}`).then(r => r.json()),
};

// Usage: swap implementations without changing business logic
const todoService = createTodoService(localStorageAdapter);
```

## DRY — Don't Repeat Yourself

### The Rule of Three

**Don't abstract too early.** Duplication is far cheaper than the wrong abstraction.

1. **First time:** Just write the code
2. **Second time:** Note the duplication, but keep it inline
3. **Third time:** Now consider extracting — you have enough examples to find the right abstraction

### The Wrong Abstraction

A bad abstraction is worse than duplication. Signs you've over-abstracted:
- The shared function has many boolean parameters or flags
- You're adding `if/else` branches for each new consumer
- Callers pass unused parameters as `null` or `undefined`
- Changing the shared code for one use case breaks another

**When to un-DRY:** If a shared function grows conditional branches to serve different callers, inline it back into each caller and let them diverge. Parallel code that evolves independently is not duplication — it's coincidence.

### What to Share vs. Not

**Share:**
- Pure utility functions (formatting, validation, math)
- Design tokens and constants
- Type definitions / schemas
- API client configuration

**Don't share prematurely:**
- UI component internals between unrelated features
- Business logic across different domains
- Test utilities used by only one test file

## Naming Conventions

### Functions
- **Verb + noun:** `getUserById`, `validateEmail`, `formatCurrency`
- **Boolean returns:** `is*`, `has*`, `can*`, `should*` — `isValid`, `hasPermission`
- **Event handlers:** `handle*` or `on*` — `handleClick`, `onSubmit`
- **Async functions:** describe what they return, not that they're async — `fetchUser` not `asyncGetUser`

### Variables
- **Booleans:** `is*`, `has*`, `should*` — never naked adjectives (`visible` → `isVisible`)
- **Arrays:** plural nouns — `users`, `items`, `errors`
- **Counts:** `*Count` — `errorCount`, `retryCount`
- **Collections/Maps:** describe the key-value relationship — `usersByID`, `priceByProduct`

### Constants
- **Uppercase with underscores for true constants:** `MAX_RETRIES`, `API_BASE_URL`
- **Regular camelCase for configuration objects:** `defaultOptions`, `routeConfig`

### Files
- Match the primary export name
- Components: PascalCase (`UserProfile.tsx`)
- Utilities/hooks: camelCase (`useAuth.ts`, `formatDate.ts`)
- Constants/config: camelCase (`apiConfig.ts`, `routes.ts`)
- Tests: match source file + `.test` suffix (`UserProfile.test.tsx`)

## Function Design

### Size
- A function should fit on one screen (~25-40 lines)
- If you need to scroll to understand it, it's too long
- Extract named helper functions for distinct logical steps

### Parameters
- **0-2 parameters:** ideal
- **3 parameters:** acceptable
- **4+ parameters:** use an options object
- **Boolean parameters:** avoid — they make call sites unreadable. Use named options or separate functions

```javascript
// Bad: what does `true` mean?
renderList(items, true, false);

// Good: named options
renderList(items, { sortable: true, filterable: false });

// Better if only one behavior differs: separate functions
renderSortableList(items);
```

### Return Values
- Return early for guard clauses — avoid deep nesting
- Be consistent: if a function can fail, always return the same shape (result or error)
- Avoid returning `null` when an empty collection (`[]` or `{}`) would work

## Error Handling

- Throw errors at system boundaries (API calls, user input, file I/O)
- Don't catch errors you can't handle — let them propagate
- Error messages should describe what went wrong AND what the user/developer should do
- Never silently swallow errors with empty `catch` blocks
- Use typed/custom errors when callers need to distinguish error types

```javascript
// Bad: swallows all errors
try { await saveData(); } catch (e) {}

// Bad: generic message
throw new Error('Something went wrong');

// Good: actionable message
throw new Error(`Failed to save user ${userId}: server returned ${response.status}. Check API connectivity.`);
```

## File Organization

```
feature/
  components/       # UI components
  hooks/            # Custom hooks (if applicable)
  utils/            # Pure helper functions
  types.ts          # Type definitions (if applicable)
  constants.ts      # Feature-specific constants
  index.ts          # Public API (barrel file)
```

**Rules:**
- Group by feature, not by file type (not `components/`, `hooks/`, `utils/` at root)
- Barrel files (`index.ts`) export only the public API — not internal helpers
- Keep import paths shallow — if you're importing `../../../utils`, the file is in the wrong place
- Co-locate tests with source files, or in a `__tests__` directory next to the source

## Comments Policy

**Don't comment what, comment why.**

- No comments restating the code: `// increment counter` above `counter++`
- No commented-out code — delete it (git remembers)
- DO comment: business rules, workarounds, non-obvious constraints
- DO comment: links to issues/specs for complex logic
- DO add JSDoc for public API functions with non-obvious parameters

```javascript
// Bad
// Check if user is admin
if (user.role === 'admin') { ... }

// Good
// Admins bypass the approval workflow per JIRA-1234
if (user.role === 'admin') { ... }
```

## Immutability & State

- Prefer `const` over `let` — use `let` only when reassignment is truly needed
- Don't mutate function arguments — return new values
- Prefer spreading/mapping over push/splice for array operations
- Prefer object spread over property assignment for objects
- Keep state as close to where it's used as possible — don't hoist state to global/parent prematurely

## Code Review Checklist

When reviewing code, check for:

1. **Can I understand what this does in 10 seconds?** If not, it needs better naming or structure
2. **Does each function do one thing?** Check for "and" in the name or description
3. **Are there magic numbers/strings?** Extract to named constants
4. **Is there duplicated logic (3+ occurrences)?** Consider extracting
5. **Are error cases handled?** Especially at system boundaries
6. **Are names accurate?** A function called `getUser` shouldn't modify state
7. **Is the abstraction level consistent?** Don't mix high-level orchestration with low-level details
8. **Can I test this in isolation?** If not, dependencies need to be injectable
9. **Are there unnecessary comments?** Remove comments that restate the code
10. **Is there dead code?** Remove unused functions, variables, imports

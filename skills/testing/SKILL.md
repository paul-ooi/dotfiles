---
triggers:
  - test
  - testing
  - vitest
  - playwright
  - fixture
  - mock
  - test suite
  - unit test
  - e2e
  - integration test
  - test setup
  - spec
---

# Testing Skill — Vitest & Playwright

This skill provides test strategy, patterns, and tooling guidance for web projects using Vitest (unit/integration) and Playwright (e2e/a11y). Framework-agnostic testing principles.

## Testing Philosophy

### Test Behavior, Not Implementation

Tests should verify **what** a component/function does, not **how** it does it internally.

```javascript
// Bad: testing implementation details
expect(component.state.isOpen).toBe(true);
expect(spy).toHaveBeenCalledTimes(1);

// Good: testing observable behavior
expect(screen.getByRole('dialog')).toBeVisible();
expect(screen.getByText('Success')).toBeInTheDocument();
```

### Arrange-Act-Assert

Every test follows this structure:

```javascript
it('shows error when email is invalid', () => {
  // Arrange: set up preconditions
  const form = renderForm();

  // Act: perform the action
  form.fillEmail('not-an-email');
  form.submit();

  // Assert: verify the outcome
  expect(form.getError()).toBe('Please enter a valid email');
});
```

### One Assertion Concept Per Test

Each test verifies one behavior. Multiple `expect` calls are fine if they verify the same concept:

```javascript
// Good: multiple expects, one concept (dialog opens correctly)
it('opens dialog with correct content', () => {
  clickTrigger();
  expect(screen.getByRole('dialog')).toBeVisible();
  expect(screen.getByText('Confirm Action')).toBeInTheDocument();
  expect(screen.getByRole('button', { name: 'Cancel' })).toHaveFocus();
});

// Bad: multiple unrelated concepts in one test
it('dialog works', () => {
  clickTrigger();
  expect(dialog).toBeVisible();         // Opening behavior
  pressEscape();
  expect(dialog).not.toBeVisible();     // Closing behavior — separate test
});
```

### Tests as Documentation

Test descriptions should read like specifications:

```javascript
describe('UserLogin', () => {
  it('redirects to dashboard after successful login', () => { ... });
  it('shows error message when credentials are invalid', () => { ... });
  it('disables submit button while request is pending', () => { ... });
  it('preserves email input when login fails', () => { ... });
});
```

## Fixture & Mock Management

### Always Check for Existing Fixtures First

Before creating inline test data or mocks:

1. **Search for existing fixtures** in `__mocks__/`, `test/fixtures/`, `test/helpers/`, or `*.factory.*` files
2. **Extend existing fixtures** rather than duplicating — use factory override patterns
3. **Create new shared fixtures** when a pattern appears 2+ times

### Factory Pattern

```typescript
// test/fixtures/user.factory.ts
interface UserOverrides {
  id?: string;
  name?: string;
  email?: string;
  role?: 'admin' | 'member' | 'viewer';
}

export function createUser(overrides: UserOverrides = {}): User {
  return {
    id: overrides.id ?? crypto.randomUUID(),
    name: overrides.name ?? 'Test User',
    email: overrides.email ?? 'test@example.com',
    role: overrides.role ?? 'member',
    createdAt: new Date().toISOString(),
    ...overrides,
  };
}

// Usage in tests
const admin = createUser({ role: 'admin' });
const viewer = createUser({ role: 'viewer', name: 'View Only' });
```

### Mock Organization

```
test/
  fixtures/
    user.factory.ts      # Data factories
    api-responses.ts      # Common API response shapes
  helpers/
    render.ts             # Custom render with providers
    setup.ts              # Global test setup
__mocks__/
  api-client.ts           # Module mock for API client
  storage.ts              # Module mock for storage
```

**Rules:**
- Centralize mocks — avoid duplicating mock implementations across test files
- Factories for test data — never hardcode objects inline when a factory exists
- Avoid over-mocking — use real implementations when they're fast and deterministic
- Reset state between tests — `beforeEach`/`afterEach` for cleanup

### When to Mock

| Mock | Don't mock |
|---|---|
| Network requests (API calls) | Pure utility functions |
| Browser APIs not in jsdom (IntersectionObserver) | Components you own |
| Timers (`vi.useFakeTimers`) | DOM interactions |
| External services | Framework primitives |
| Randomness/dates when determinism needed | Internal module dependencies (usually) |

## Vitest — Unit & Integration Testing

### Test Organization

```typescript
describe('formatCurrency', () => {
  it('formats positive amounts with two decimal places', () => {
    expect(formatCurrency(10)).toBe('$10.00');
  });

  it('formats negative amounts with minus sign', () => {
    expect(formatCurrency(-5.5)).toBe('-$5.50');
  });

  it('returns $0.00 for zero', () => {
    expect(formatCurrency(0)).toBe('$0.00');
  });
});
```

### Setup & Teardown

```typescript
describe('UserService', () => {
  let service: UserService;

  beforeEach(() => {
    service = new UserService(mockApiClient);
    vi.clearAllMocks();
  });

  afterAll(() => {
    vi.restoreAllMocks();
  });
});
```

### Module Mocking

```typescript
// Mock an entire module
vi.mock('./api-client', () => ({
  fetchUser: vi.fn(),
  updateUser: vi.fn(),
}));

// Mock with factory (auto-mock everything, override specific exports)
vi.mock('./api-client', async (importOriginal) => {
  const actual = await importOriginal();
  return {
    ...actual,
    fetchUser: vi.fn(),
  };
});

// Inline mock for a specific test
import { fetchUser } from './api-client';

it('handles API errors', async () => {
  vi.mocked(fetchUser).mockRejectedValue(new Error('Network error'));
  // ...test error handling
});
```

### Spies

```typescript
const onSubmit = vi.fn();
renderForm({ onSubmit });
submitButton.click();
expect(onSubmit).toHaveBeenCalledWith({ email: 'test@example.com' });
```

### Snapshot Testing

**Use sparingly.** Snapshots are appropriate for:
- Serialized data structures (API response transformations)
- Error messages / formatted output

**Don't use for:**
- Component rendering (too fragile, meaningless diffs)
- Large objects (hard to review in PRs)

```typescript
// Good: small, meaningful snapshot
expect(formatErrorMessages(validationResult)).toMatchInlineSnapshot(`
  [
    "Email is required",
    "Password must be at least 8 characters",
  ]
`);
```

### Coverage

Aim for meaningful coverage, not 100%:
- **Unit utilities:** 95%+ is reasonable — they're pure functions
- **Components:** focus on behavior coverage, not line coverage
- **Integration tests:** cover happy path + main error paths
- **Don't test:** framework boilerplate, type-only code, trivial getters

## Playwright — E2E & Accessibility Testing

### Page Object Model

```typescript
// pages/login.page.ts
export class LoginPage {
  constructor(private page: Page) {}

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.page.getByLabel('Email').fill(email);
    await this.page.getByLabel('Password').fill(password);
    await this.page.getByRole('button', { name: 'Sign in' }).click();
  }

  getError() {
    return this.page.getByRole('alert');
  }
}

// tests/login.spec.ts
test('successful login redirects to dashboard', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login('user@example.com', 'password123');
  await expect(page).toHaveURL('/dashboard');
});
```

### Shared Fixtures with test.extend

```typescript
// fixtures.ts
import { test as base } from '@playwright/test';
import { LoginPage } from './pages/login.page';
import { DashboardPage } from './pages/dashboard.page';

export const test = base.extend<{
  loginPage: LoginPage;
  dashboardPage: DashboardPage;
}>({
  loginPage: async ({ page }, use) => {
    await use(new LoginPage(page));
  },
  dashboardPage: async ({ page }, use) => {
    await use(new DashboardPage(page));
  },
});

// Usage
test('dashboard shows user name', async ({ loginPage, dashboardPage }) => {
  await loginPage.goto();
  await loginPage.login('user@example.com', 'password123');
  await expect(dashboardPage.greeting).toContainText('Welcome');
});
```

### Locator Strategy — Accessible Selectors First

**Prefer accessible selectors** (they verify accessibility AND locate elements):

```typescript
// Best: accessible selectors (verify a11y as a side effect)
page.getByRole('button', { name: 'Submit' });
page.getByLabel('Email address');
page.getByRole('heading', { name: 'Settings' });
page.getByRole('navigation');
page.getByRole('alert');

// Acceptable: text content
page.getByText('Welcome back');
page.getByPlaceholder('Search...');

// Last resort: test IDs (when no accessible selector works)
page.getByTestId('complex-widget');

// Avoid: CSS selectors (brittle, no a11y benefit)
page.locator('.btn-primary');
page.locator('#submit-button');
```

### Accessibility Testing with @axe-core/playwright

```typescript
import AxeBuilder from '@axe-core/playwright';

test('home page has no accessibility violations', async ({ page }) => {
  await page.goto('/');
  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa', 'wcag22aa'])
    .analyze();
  expect(results.violations).toEqual([]);
});

// Test after interaction
test('modal is accessible when open', async ({ page }) => {
  await page.goto('/');
  await page.getByRole('button', { name: 'Open settings' }).click();
  await expect(page.getByRole('dialog')).toBeVisible();

  const results = await new AxeBuilder({ page })
    .include('[role="dialog"]')
    .withTags(['wcag2a', 'wcag2aa', 'wcag22aa'])
    .analyze();
  expect(results.violations).toEqual([]);
});
```

### Visual Regression

```typescript
test('landing page matches snapshot', async ({ page }) => {
  await page.goto('/');
  await expect(page).toHaveScreenshot('landing.png', {
    maxDiffPixelRatio: 0.01,
  });
});

// Component-level
test('button variants render correctly', async ({ page }) => {
  await page.goto('/storybook/button');
  await expect(page.getByTestId('button-group')).toHaveScreenshot('buttons.png');
});
```

## Test Suite Setup

When establishing a new test suite for a project, scaffold the infrastructure first:

### 1. Project Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./test/setup.ts'],
    include: ['src/**/*.test.{ts,tsx}'],
    coverage: {
      provider: 'v8',
      include: ['src/**'],
      exclude: ['src/**/*.test.*', 'src/**/index.ts'],
    },
  },
});
```

### 2. Global Setup

```typescript
// test/setup.ts
import '@testing-library/jest-dom/vitest';

// Global mocks
vi.mock('./src/config', () => ({
  API_URL: 'http://localhost:3000',
}));

// Reset between tests
afterEach(() => {
  vi.clearAllMocks();
});
```

### 3. Custom Render (if using a UI framework with providers)

```typescript
// test/helpers/render.ts
function customRender(ui: ReactElement, options?: RenderOptions) {
  return render(ui, {
    wrapper: ({ children }) => (
      <ThemeProvider>
        <RouterProvider>
          {children}
        </RouterProvider>
      </ThemeProvider>
    ),
    ...options,
  });
}

export { customRender as render };
```

### 4. Playwright Config

```typescript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  baseURL: 'http://localhost:3000',
  use: {
    screenshot: 'only-on-failure',
    trace: 'retain-on-failure',
  },
  projects: [
    { name: 'chromium', use: { browserName: 'chromium' } },
    { name: 'firefox', use: { browserName: 'firefox' } },
    { name: 'webkit', use: { browserName: 'webkit' } },
  ],
});
```

## Browser Verification with MCP Tools

When developing tests, use chrome-devtools MCP tools for interactive verification:

### Verify Rendered State
Use `take_snapshot` to inspect what the test should assert:
- Check that elements have expected roles and names
- Verify ARIA states match expected test state
- Confirm content is visible/hidden as expected

### Inspect DOM State
Use `evaluate_script` to examine state that tests should verify:
```javascript
// Check focused element
() => document.activeElement?.tagName + '#' + document.activeElement?.id

// Check form validity
() => document.querySelector('form').checkValidity()

// Check computed styles
(el) => getComputedStyle(el).display
```

### Check for Console Errors
Use `list_console_messages` to find errors/warnings tests should catch:
- React warnings about invalid props
- Unhandled promise rejections
- Missing resource errors

## MCP Tool Integration

When available, use specialized MCP tools:

### Playwright MCP (when available)
- Run specific test suites or individual tests
- Manage browser contexts for e2e testing
- Execute Playwright test commands

### Vitest MCP (when available)
- Run unit/integration test suites
- Watch mode for active development
- Check coverage reports

## Code Review Checklist for Tests

1. **Does the test describe behavior?** Not implementation details
2. **Is the test independent?** No reliance on test execution order
3. **Are fixtures shared?** Check for existing factories before creating inline data
4. **Is the mock minimal?** Only mock what's necessary
5. **Are accessible selectors used?** `getByRole` over `getByTestId` over CSS selectors
6. **Is there proper cleanup?** Mocks reset, side effects reversed
7. **Does the description read like a spec?** `it('shows error when...')` not `it('test error')`
8. **Are edge cases covered?** Empty states, error states, loading states
9. **Is the test deterministic?** No flaky timing, no random data without seeds
10. **Are a11y checks included?** axe-core assertions in e2e tests

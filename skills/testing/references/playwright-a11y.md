# Playwright Accessibility Testing

## Setup

Install `@axe-core/playwright`:

```bash
npm install -D @axe-core/playwright
```

## Full Page Accessibility Test

```typescript
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('accessibility', () => {
  test('home page has no violations', async ({ page }) => {
    await page.goto('/');

    const results = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag22aa'])
      .analyze();

    expect(results.violations).toEqual([]);
  });
});
```

## Scoped Accessibility Test

Test a specific part of the page:

```typescript
test('navigation is accessible', async ({ page }) => {
  await page.goto('/');

  const results = await new AxeBuilder({ page })
    .include('nav')
    .withTags(['wcag2a', 'wcag2aa', 'wcag22aa'])
    .analyze();

  expect(results.violations).toEqual([]);
});
```

## Test After Interaction

Accessibility issues often appear only after user interaction (modals, menus, error states):

```typescript
test('login form shows accessible error messages', async ({ page }) => {
  await page.goto('/login');

  // Submit empty form to trigger errors
  await page.getByRole('button', { name: 'Sign in' }).click();

  // Check accessibility of error state
  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa', 'wcag22aa'])
    .analyze();

  expect(results.violations).toEqual([]);

  // Also verify error messages are associated with inputs
  const emailInput = page.getByLabel('Email');
  await expect(emailInput).toHaveAttribute('aria-invalid', 'true');
  await expect(emailInput).toHaveAttribute('aria-describedby');
});

test('modal dialog is accessible when open', async ({ page }) => {
  await page.goto('/');

  // Open the modal
  await page.getByRole('button', { name: 'Delete account' }).click();
  const dialog = page.getByRole('dialog');
  await expect(dialog).toBeVisible();

  // Check the dialog specifically
  const results = await new AxeBuilder({ page })
    .include('[role="dialog"]')
    .withTags(['wcag2a', 'wcag2aa', 'wcag22aa'])
    .analyze();

  expect(results.violations).toEqual([]);

  // Verify focus is inside dialog
  const focused = await page.evaluate(() => {
    const dialog = document.querySelector('[role="dialog"]');
    return dialog?.contains(document.activeElement);
  });
  expect(focused).toBe(true);
});
```

## Keyboard Navigation Tests

```typescript
test('tab navigation follows logical order', async ({ page }) => {
  await page.goto('/');

  // Tab through elements, verify order
  await page.keyboard.press('Tab');
  await expect(page.getByRole('link', { name: 'Skip to content' })).toBeFocused();

  await page.keyboard.press('Tab');
  await expect(page.getByRole('link', { name: 'Home' })).toBeFocused();

  await page.keyboard.press('Tab');
  await expect(page.getByRole('link', { name: 'About' })).toBeFocused();
});

test('dropdown menu keyboard navigation', async ({ page }) => {
  await page.goto('/');

  const trigger = page.getByRole('button', { name: 'Actions' });
  await trigger.focus();

  // Open with Enter
  await page.keyboard.press('Enter');
  await expect(page.getByRole('menu')).toBeVisible();

  // Navigate with arrows
  await page.keyboard.press('ArrowDown');
  await expect(page.getByRole('menuitem', { name: 'Edit' })).toBeFocused();

  await page.keyboard.press('ArrowDown');
  await expect(page.getByRole('menuitem', { name: 'Delete' })).toBeFocused();

  // Close with Escape
  await page.keyboard.press('Escape');
  await expect(page.getByRole('menu')).not.toBeVisible();
  await expect(trigger).toBeFocused(); // Focus returns to trigger
});

test('modal traps focus', async ({ page }) => {
  await page.goto('/');

  await page.getByRole('button', { name: 'Open settings' }).click();
  const dialog = page.getByRole('dialog');
  await expect(dialog).toBeVisible();

  // Tab through all focusable elements in dialog
  const focusableElements = await dialog.locator(
    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
  ).all();

  // Tab past the last element should cycle back to first
  for (const el of focusableElements) {
    await page.keyboard.press('Tab');
  }
  await page.keyboard.press('Tab');

  // Focus should still be inside dialog
  const focusInDialog = await page.evaluate(() => {
    const dialog = document.querySelector('[role="dialog"]');
    return dialog?.contains(document.activeElement);
  });
  expect(focusInDialog).toBe(true);
});
```

## Color Scheme Testing

```typescript
test('dark mode has no accessibility violations', async ({ page }) => {
  await page.emulateMedia({ colorScheme: 'dark' });
  await page.goto('/');

  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa', 'wcag22aa'])
    .analyze();

  expect(results.violations).toEqual([]);
});

test('light mode has no accessibility violations', async ({ page }) => {
  await page.emulateMedia({ colorScheme: 'light' });
  await page.goto('/');

  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa', 'wcag22aa'])
    .analyze();

  expect(results.violations).toEqual([]);
});
```

## Reduced Motion Testing

```typescript
test('content is accessible with reduced motion', async ({ page }) => {
  await page.emulateMedia({ reducedMotion: 'reduce' });
  await page.goto('/');

  // Verify animations are disabled
  const hasAnimations = await page.evaluate(() => {
    const el = document.querySelector('.animated-element');
    if (!el) return false;
    const style = getComputedStyle(el);
    return style.animationDuration !== '0s' && style.animationDuration !== '0.01ms';
  });

  expect(hasAnimations).toBe(false);
});
```

## Responsive Accessibility

```typescript
const viewports = [
  { width: 320, height: 568, name: 'mobile' },
  { width: 768, height: 1024, name: 'tablet' },
  { width: 1280, height: 800, name: 'desktop' },
];

for (const viewport of viewports) {
  test(`${viewport.name} viewport has no accessibility violations`, async ({ page }) => {
    await page.setViewportSize({ width: viewport.width, height: viewport.height });
    await page.goto('/');

    const results = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag22aa'])
      .analyze();

    expect(results.violations).toEqual([]);
  });
}
```

## Accessibility Test Fixture

Create a reusable fixture for axe checks:

```typescript
// fixtures/a11y.ts
import { test as base, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

export const test = base.extend<{
  checkA11y: (selector?: string) => Promise<void>;
}>({
  checkA11y: async ({ page }, use) => {
    const checkA11y = async (selector?: string) => {
      let builder = new AxeBuilder({ page })
        .withTags(['wcag2a', 'wcag2aa', 'wcag22aa']);

      if (selector) {
        builder = builder.include(selector);
      }

      const results = await builder.analyze();
      expect(results.violations).toEqual([]);
    };
    await use(checkA11y);
  },
});

// Usage
test('page is accessible', async ({ page, checkA11y }) => {
  await page.goto('/');
  await checkA11y();
});

test('dialog is accessible', async ({ page, checkA11y }) => {
  await page.goto('/');
  await page.getByRole('button', { name: 'Open' }).click();
  await checkA11y('[role="dialog"]');
});
```

## Reporting Violations

Helper to format violation output for readable test failures:

```typescript
function formatViolations(violations: AxeResults['violations']): string {
  return violations
    .map((v) => {
      const nodes = v.nodes.map((n) => `  - ${n.html}\n    ${n.failureSummary}`).join('\n');
      return `[${v.impact}] ${v.id}: ${v.help}\n${nodes}`;
    })
    .join('\n\n');
}

// Usage in assertion
const results = await new AxeBuilder({ page }).analyze();
expect(
  results.violations,
  formatViolations(results.violations)
).toEqual([]);
```

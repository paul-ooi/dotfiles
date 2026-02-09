# Fixture & Mock Patterns

## Contents
- Factory Functions (Basic, Related Entity, Builder Pattern)
- API Response Fixtures
- Mock Implementations (API Client, Storage, Timer Control)
- Custom Matchers
- Test Data Principles

## Factory Functions

### Basic Factory

```typescript
// test/fixtures/user.factory.ts
let idCounter = 0;

export function createUser(overrides: Partial<User> = {}): User {
  idCounter++;
  return {
    id: `user-${idCounter}`,
    name: 'Test User',
    email: `user-${idCounter}@example.com`,
    role: 'member',
    createdAt: '2024-01-01T00:00:00Z',
    avatar: null,
    ...overrides,
  };
}

// Reset counter between tests if needed
export function resetUserFactory() {
  idCounter = 0;
}
```

### Related Entity Factory

```typescript
// test/fixtures/project.factory.ts
import { createUser } from './user.factory';

export function createProject(overrides: Partial<Project> = {}): Project {
  return {
    id: crypto.randomUUID(),
    name: 'Test Project',
    owner: createUser(),
    members: [],
    status: 'active',
    createdAt: '2024-01-01T00:00:00Z',
    ...overrides,
  };
}

// Compose: project with members
export function createProjectWithMembers(memberCount = 3): Project {
  return createProject({
    members: Array.from({ length: memberCount }, () => createUser()),
  });
}
```

### Builder Pattern (For Complex Objects)

```typescript
class UserBuilder {
  private data: Partial<User> = {};

  withRole(role: User['role']) {
    this.data.role = role;
    return this;
  }

  withName(name: string) {
    this.data.name = name;
    return this;
  }

  asAdmin() {
    this.data.role = 'admin';
    this.data.permissions = ['read', 'write', 'delete', 'admin'];
    return this;
  }

  build(): User {
    return createUser(this.data);
  }
}

export const userBuilder = () => new UserBuilder();

// Usage
const admin = userBuilder().asAdmin().withName('Admin User').build();
```

## API Response Fixtures

```typescript
// test/fixtures/api-responses.ts

export function createApiResponse<T>(data: T, overrides?: Partial<ApiResponse<T>>): ApiResponse<T> {
  return {
    data,
    status: 200,
    message: 'OK',
    timestamp: '2024-01-01T00:00:00Z',
    ...overrides,
  };
}

export function createPaginatedResponse<T>(
  items: T[],
  overrides?: Partial<PaginatedResponse<T>>
): PaginatedResponse<T> {
  return {
    data: items,
    total: items.length,
    page: 1,
    pageSize: 20,
    hasMore: false,
    ...overrides,
  };
}

export function createApiError(
  status: number,
  message: string,
  overrides?: Partial<ApiError>
): ApiError {
  return {
    status,
    message,
    code: `ERR_${status}`,
    timestamp: '2024-01-01T00:00:00Z',
    ...overrides,
  };
}
```

## Mock Implementations

### API Client Mock

```typescript
// __mocks__/api-client.ts
import { vi } from 'vitest';

export const apiClient = {
  get: vi.fn(),
  post: vi.fn(),
  put: vi.fn(),
  delete: vi.fn(),
};

// Helper to set up common responses
export function mockApiSuccess<T>(method: 'get' | 'post' | 'put' | 'delete', data: T) {
  apiClient[method].mockResolvedValue({ data, status: 200 });
}

export function mockApiError(method: 'get' | 'post' | 'put' | 'delete', status: number, message: string) {
  apiClient[method].mockRejectedValue({ status, message });
}
```

### Storage Mock

```typescript
// __mocks__/storage.ts
export function createStorageMock(): Storage {
  const store = new Map<string, string>();
  return {
    getItem: (key: string) => store.get(key) ?? null,
    setItem: (key: string, value: string) => store.set(key, value),
    removeItem: (key: string) => store.delete(key),
    clear: () => store.clear(),
    get length() { return store.size; },
    key: (index: number) => [...store.keys()][index] ?? null,
  };
}
```

### Timer Control

```typescript
// In tests that depend on time
beforeEach(() => {
  vi.useFakeTimers();
});

afterEach(() => {
  vi.useRealTimers();
});

it('debounces search input', async () => {
  renderSearchInput();
  await userEvent.type(screen.getByRole('searchbox'), 'hello');

  // Not called yet (debounce active)
  expect(onSearch).not.toHaveBeenCalled();

  // Advance past debounce delay
  vi.advanceTimersByTime(300);

  expect(onSearch).toHaveBeenCalledWith('hello');
});
```

## Custom Matchers

```typescript
// test/helpers/matchers.ts
import { expect } from 'vitest';

expect.extend({
  toBeWithinRange(received: number, floor: number, ceiling: number) {
    const pass = received >= floor && received <= ceiling;
    return {
      pass,
      message: () =>
        `expected ${received} ${pass ? 'not ' : ''}to be within range ${floor}–${ceiling}`,
    };
  },

  toHaveAccessibleName(received: HTMLElement, name: string) {
    const accessibleName = received.getAttribute('aria-label')
      || received.getAttribute('aria-labelledby')
      || received.textContent;
    const pass = accessibleName?.includes(name) ?? false;
    return {
      pass,
      message: () =>
        `expected element ${pass ? 'not ' : ''}to have accessible name "${name}", got "${accessibleName}"`,
    };
  },
});
```

## Test Data Principles

1. **Minimal data** — only include fields the test cares about; factory provides sensible defaults
2. **Unique identifiers** — use counters or UUID to avoid collisions between tests
3. **Deterministic** — no `Math.random()` or `Date.now()` in factories without seeding
4. **Readable** — override values should make the test's intent clear

```typescript
// Good: overrides tell you what matters
const expiredUser = createUser({ status: 'expired', expiresAt: '2020-01-01' });

// Bad: inline object with lots of irrelevant fields
const expiredUser = { id: '1', name: 'Test', email: 'a@b.com', role: 'member', status: 'expired', ... };
```

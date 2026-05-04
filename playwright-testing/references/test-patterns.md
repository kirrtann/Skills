# Playwright Test Patterns — Pro Level

## Selector priority (best → worst)

```ts
getByRole('button', { name: 'Submit' }); // ✅ best — semantic
getByLabel('Email address'); // ✅ best — form fields
getByPlaceholder('you@example.com'); // ✅ good
getByText('Welcome back'); // ✅ good for visible text
getByTestId('submit-btn'); // ✅ good — explicit test id
locator('.submit-btn'); // ⚠️  avoid CSS unless no choice
locator('#submit'); // ⚠️  avoid ID selectors
```

---

## Auth Tests — `tests/auth.spec.ts`

```ts
import { test, expect } from '@playwright/test';

const BASE = '/';
const CREDS = { email: 'test@example.com', password: 'Password123!' };

test.describe('Authentication', () => {
  test('login with valid credentials', async ({ page }) => {
    await page.goto('/login');
    await page.getByLabel('Email').fill(CREDS.email);
    await page.getByLabel('Password').fill(CREDS.password);
    await page.getByRole('button', { name: 'Sign in' }).click();
    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByRole('heading', { name: /dashboard/i })).toBeVisible();
  });

  test('shows error on invalid credentials', async ({ page }) => {
    await page.goto('/login');
    await page.getByLabel('Email').fill('wrong@example.com');
    await page.getByLabel('Password').fill('wrongpass');
    await page.getByRole('button', { name: 'Sign in' }).click();
    await expect(page.getByText(/invalid email or password/i)).toBeVisible();
  });

  test('redirects unauthenticated user from protected route', async ({ page }) => {
    await page.goto('/dashboard');
    await expect(page).toHaveURL('/login');
  });

  test('logout clears session', async ({ page }) => {
    // Login first
    await page.goto('/login');
    await page.getByLabel('Email').fill(CREDS.email);
    await page.getByLabel('Password').fill(CREDS.password);
    await page.getByRole('button', { name: 'Sign in' }).click();
    await expect(page).toHaveURL('/dashboard');
    // Logout
    await page.getByRole('button', { name: /logout/i }).click();
    await expect(page).toHaveURL('/login');
    // Confirm session cleared
    await page.goto('/dashboard');
    await expect(page).toHaveURL('/login');
  });
});
```

---

## Form Tests — `tests/[feature]-form.spec.ts`

```ts
import { test, expect } from '@playwright/test';

test.describe('Contact Form', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/contact');
  });

  test('submits form with valid data', async ({ page }) => {
    await page.getByLabel('Name').fill('John Doe');
    await page.getByLabel('Email').fill('john@example.com');
    await page.getByLabel('Message').fill('Hello, this is a test message.');
    await page.getByRole('button', { name: 'Send' }).click();
    await expect(page.getByText(/message sent/i)).toBeVisible();
  });

  test('shows validation errors on empty submit', async ({ page }) => {
    await page.getByRole('button', { name: 'Send' }).click();
    await expect(page.getByText(/name is required/i)).toBeVisible();
    await expect(page.getByText(/email is required/i)).toBeVisible();
  });

  test('shows error for invalid email format', async ({ page }) => {
    await page.getByLabel('Email').fill('not-an-email');
    await page.getByRole('button', { name: 'Send' }).click();
    await expect(page.getByText(/valid email/i)).toBeVisible();
  });
});
```

---

## Navigation Tests — `tests/navigation.spec.ts`

```ts
import { test, expect } from '@playwright/test';

const ROUTES = [
  { path: '/', title: /home/i },
  { path: '/about', title: /about/i },
  { path: '/dashboard', title: /dashboard/i },
];

test.describe('Navigation', () => {
  for (const route of ROUTES) {
    test(`${route.path} loads correctly`, async ({ page }) => {
      await page.goto(route.path);
      await expect(page.getByRole('heading', { name: route.title })).toBeVisible();
    });
  }

  test('404 page shown for unknown route', async ({ page }) => {
    await page.goto('/this-does-not-exist');
    await expect(page.getByText(/404|not found/i)).toBeVisible();
  });

  test('nav links are clickable', async ({ page }) => {
    await page.goto('/');
    await page.getByRole('link', { name: /about/i }).click();
    await expect(page).toHaveURL('/about');
  });
});
```

---

## Data Table / List Tests — `tests/[feature]-table.spec.ts`

```ts
import { test, expect } from '@playwright/test';

test.describe('Users Table', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/users');
  });

  test('renders table with data', async ({ page }) => {
    await expect(page.getByRole('table')).toBeVisible();
    const rows = page.getByRole('row');
    await expect(rows).toHaveCount({ minimum: 2 }); // header + at least 1 data row
  });

  test('shows empty state when no data', async ({ page }) => {
    // Intercept API to return empty
    await page.route('**/api/users', (route) => route.fulfill({ json: [] }));
    await page.reload();
    await expect(page.getByText(/no data/i)).toBeVisible();
  });

  test('search filters results', async ({ page }) => {
    await page.getByPlaceholder('Search').fill('John');
    await expect(page.getByText('John Doe')).toBeVisible();
  });
});
```

---

## API Mock Pattern

```ts
// Mock a REST API response
await page.route('**/api/dashboard/stats', (route) =>
  route.fulfill({
    status: 200,
    contentType: 'application/json',
    json: { totalUsers: 42, revenue: 9800, growth: 12.5 },
  })
);

// Mock an error response
await page.route('**/api/users', (route) =>
  route.fulfill({ status: 500, body: 'Internal Server Error' })
);
```

---

## Auth State Reuse (avoid logging in for every test)

```ts
// tests/auth.setup.ts
import { test as setup } from '@playwright/test';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill('test@example.com');
  await page.getByLabel('Password').fill('Password123!');
  await page.getByRole('button', { name: 'Sign in' }).click();
  await page.waitForURL('/dashboard');
  await page.context().storageState({ path: 'tests/.auth/user.json' });
});
```

```ts
// playwright.config.ts — add to projects:
{
  name: 'setup',
  testMatch: /auth\.setup\.ts/,
},
{
  name: 'authenticated',
  use: { storageState: 'tests/.auth/user.json' },
  dependencies: ['setup'],
},
```

---

## Key Rules for Writing Tests

- Always use `await expect(locator).toBeVisible()` — never `waitForTimeout()`
- Use `test.beforeEach` to avoid repeating `page.goto()`
- Group related tests in `test.describe()`
- Keep test data as constants at top of file
- One assertion per logical outcome — don't chain unrelated assertions
- Use `page.route()` to mock APIs — don't depend on real backend for unit-level E2E

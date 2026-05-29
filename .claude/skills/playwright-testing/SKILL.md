---
name: playwright-testing
description: >
  Pro-level Playwright end-to-end testing setup and test writing for React/Next.js/Vite projects.
  Handles full setup from scratch OR writing/syncing missing test cases for already-configured
  projects. NEVER modifies existing source files — only creates new test files and config.
  Use whenever the user mentions Playwright, E2E testing, end-to-end tests, browser testing,
  automated testing, UI testing, test automation, write tests, run tests, or fixing failing tests.
  Also trigger for "set up testing", "add tests to my project", "check test coverage", or
  "my tests are failing".
---

# Playwright Testing Skill

Two workflows. Ask which applies, then follow only that path.

---

## Decision Gate — Ask First (two options only)

> **Which situation?**
>
> 1. **Set up** — Playwright not installed. Set up, write tests, run them.
> 2. **Already set up** — Playwright configured. Check and write missing test cases, then run.

---

## ⚠️ STRICT RULE — BOTH WORKFLOWS

```
NEVER modify any existing source file (components, pages, utils, configs).
ONLY create new test files and Playwright config.
Never overwrite an existing test file — only add new ones.
```

---

## Workflow 1 — Set Up

### Step 1 — Detect stack

```bash
cat package.json | grep -E '"next"|"vite"|"react"|"vue"|"scripts"' | head -20
```

### Step 2 — Install Playwright

```bash
npm init playwright@latest -- --quiet
# Choose: TypeScript, tests/ folder, add GitHub Actions: No, install browsers: Yes
```

Or manually if the user prefers:

```bash
npm install -D @playwright/test
npx playwright install
```

### Step 3 — Create config (only if `playwright.config.ts` doesn't exist)

```bash
ls playwright.config.ts 2>/dev/null || echo "safe to create"
```

Read `references/config.md` for the config template. Detect base URL from:

```bash
cat package.json | grep -E '"dev"|"start"|"preview"'
# Common: localhost:3000 (CRA/Next), localhost:5173 (Vite), localhost:4200 (Angular)
```

### Step 4 — Understand project structure

Ask user:

> "What is your `src/` structure? Share the top-level folders (pages, components, features) so I can write relevant tests."

Or read just the top level — do NOT recurse deep:

```bash
ls src/
ls src/pages/ 2>/dev/null || ls src/app/ 2>/dev/null || ls src/views/ 2>/dev/null
```

### Step 5 — Write test cases

Read `references/test-patterns.md` for pro-level test templates.

**Test file naming:** `tests/[feature].spec.ts`

**Priority order for what to test:**

1. Auth flows (login, logout, register, protected routes)
2. Core user journeys (the main thing your app does)
3. Forms (validation, submit, error states)
4. Navigation (routes load, 404 handling)
5. Critical UI states (loading, empty, error)

Write tests for each discovered feature. See `references/test-patterns.md` for patterns.

### Step 6 — Run tests and fix errors

```bash
npx playwright test --reporter=list 2>&1 | tail -50
```

**If tests fail:**

- Read the error output carefully
- Fix selector issues (prefer `getByRole`, `getByLabel`, `getByTestId` over CSS)
- Fix timing issues (use `await expect(locator).toBeVisible()` not `waitForTimeout`)
- If base URL wrong → update `playwright.config.ts` baseURL only
- Re-run after each fix: `npx playwright test --reporter=list 2>&1 | tail -30`

### Step 7 — Print summary

```
✅ playwright.config.ts created
✅ tests/auth.spec.ts          — 4 tests (login, logout, register, protected route)
✅ tests/dashboard.spec.ts     — 3 tests (loads, table renders, empty state)
✅ tests/navigation.spec.ts    — 2 tests (links work, 404 page)

▶ Run: npx playwright test
▶ UI mode: npx playwright test --ui
▶ Single file: npx playwright test tests/auth.spec.ts
```

---

## Workflow 2 — Already Set Up

### Step 1 — Ask user for paths (no scanning)

Ask:

> "Where is your `tests/` folder? And where is `playwright.config.ts`?"

Do NOT scan project. Use exactly what the user provides.

### Step 2 — Check CHANGELOG for recently added features

```bash
head -80 CHANGELOG.md 2>/dev/null
```

If CHANGELOG lists new features/pages → those are the priority for missing tests.

### Step 3 — Read existing test files (structure only, not full content)

```bash
ls [TESTS_DIR]/
```

Then for each `.spec.ts` file, read only the `test(` and `describe(` lines to map coverage:

```bash
grep -n "test\(\|describe\(" [TESTS_DIR]/*.spec.ts 2>/dev/null
```

This gives full coverage map with minimal tokens.

### Step 4 — Read project structure (top level only)

```bash
ls src/pages/ 2>/dev/null || ls src/app/ 2>/dev/null || ls src/views/ 2>/dev/null
ls src/components/ 2>/dev/null | head -20
```

Do NOT read file contents — just names to find untested features.

### Step 5 — Identify gaps

Compare:

- Files/features found in `src/pages/` or `src/app/` → what pages exist
- Test names from Step 3 → what is already covered
- CHANGELOG entries → what was recently added

Report gaps:

```
📋 Coverage gaps found:
  - /settings page — no tests
  - /checkout flow — no tests
  - ProfileForm component — no tests
  - CHANGELOG: "added notifications" — no tests
```

### Step 6 — Write only missing tests

- One `.spec.ts` file per untested feature
- Never modify existing `.spec.ts` files
- Read `references/test-patterns.md` for patterns
- Match the style/conventions of existing test files

### Step 7 — Run all tests

```bash
npx playwright test --reporter=list 2>&1 | tail -60
```

**Fix failures** same as Workflow 1 Step 6.

### Step 8 — Print summary

```
✅ tests/settings.spec.ts      — 3 new tests written
✅ tests/checkout.spec.ts      — 5 new tests written
✅ tests/notifications.spec.ts — 2 new tests written (from CHANGELOG)

⏭ Skipped (already covered): auth, dashboard, navigation

▶ Run all: npx playwright test
▶ Run new only: npx playwright test tests/settings.spec.ts tests/checkout.spec.ts
```

---

## References — load only when needed

| File                          | Load when                      |
| ----------------------------- | ------------------------------ |
| `references/config.md`        | Writing `playwright.config.ts` |
| `references/test-patterns.md` | Writing any test file          |

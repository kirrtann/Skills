---
name: frontend-project-setup
description: >
  Scaffold fully custom, production-ready frontend projects from scratch — with full CI/CD hooks,
  testing, API services, environment configs, and Git quality gates. Use this skill whenever
  the user wants to set up, initialize, scaffold, or bootstrap a new frontend project — even if
  they do not say "skill". Triggers include: set up a React project, create a Next.js app,
  initialize a Vite project, new frontend project, project boilerplate, starter template,
  configure Tailwind, SCSS, Zustand, Redux, TypeScript, ESLint, Playwright, testing setup,
  Git hooks, pre-commit, pre-push, TanStack Query, API service layer, environment config,
  multi-env setup (local/stage/prod), or any combination of frontend stack choices. Also
  trigger when the user asks to add a specific tool such as adding Zustand to a project or
  setting up ESLint for React. When the user says "bypass" or "default", skip questions and
  scaffold a full Next.js 14 App Router project with the default stack immediately.
  Generates complete file trees, config files, scripts, and boilerplate code tailored to the
  user's chosen stack.
---

# Frontend Project Setup Skill

Scaffold complete, production-ready frontend projects with modern tooling, Git hooks, testing, API services, and multi-environment configs. Always generate **real files** — not just instructions.

---

## ⚡ BYPASS / DEFAULT MODE

If the user says **"bypass"**, **"default"**, **"skip"**, or **"just set it up"** — skip all questions and immediately scaffold with:

| Option | Default |
|---|---|
| Framework | **Next.js 14 App Router** |
| Styling | **Tailwind CSS + SCSS** |
| State | **Zustand + TanStack Query** |
| UI Library | **Ant Design v5** |
| Tables | **TanStack Table v8** |
| Icons | **React Icons + Ant Design Icons** |
| Language | **TypeScript (strict)** |
| Linting | **ESLint v9 flat config + Prettier** |
| Testing | **Playwright (E2E) + Vitest (unit)** |
| Git Hooks | **Husky + lint-staged (pre-commit) + pre-push test run** |
| API Layer | **Axios service files + TanStack Query hooks** |
| Environments | **local / stage / production `.env` files + npm scripts** |

Jump directly to **Step 3** with these defaults.

---

## Step 1: Gather Stack Choices

Use `ask_user_input_v0` to gather choices. Keep to 3 questions max per round. Cover:

### Q1 — Framework & Styling
- **Framework**: Next.js 14 (App Router) ★ | Next.js 14 (Pages Router) | React + Vite | Remix
- **Styling**: Tailwind CSS ★ | SCSS | Tailwind + SCSS ★ | CSS Modules | Styled Components

### Q2 — State, Data Fetching & UI
- **State Management**: Zustand ★ | Redux Toolkit | Jotai | Recoil | Context API only | None
- **Data Fetching**: TanStack Query (React Query) ★ | SWR | RTK Query | Fetch/Axios only | None
- **UI Library**: Ant Design v5 ★ | shadcn/ui | MUI | Chakra UI | Radix UI | None

### Q3 — Quality & Testing
- **Tables**: TanStack Table v8 ★ | Ant Design Table | AG Grid (Community) | React Data Grid | None
- **Icons**: React Icons ★ | Lucide React | Heroicons | Phosphor Icons | Tabler Icons | Ant Design Icons
- **Testing**: Playwright + Vitest ★ | Playwright only | Vitest + Testing Library | Cypress + Vitest | Jest + Testing Library | None
- **Git Hooks**: Husky (pre-commit + pre-push) ★ | Lefthook | Simple-git-hooks | None

### Q4 — AI Rules File
- **AI rules file name**: `CLAUDE.md` ★ | `AGENTS.md`
  - This file is placed at the project root and contains persistent AI coding rules for the project.
  - `CLAUDE.md` — used by Claude / Anthropic tooling
  - `AGENTS.md` — used by OpenAI Codex / other agents

★ = recommended default

---

## Step 2: Select Reference Files

Read **all** relevant reference files before generating anything:

| Topic | Reference File |
|---|---|
| React + Vite setup | `references/vite-react.md` |
| Next.js App Router | `references/nextjs-app.md` |
| Tailwind CSS | `references/tailwind.md` |
| SCSS | `references/scss.md` |
| Zustand | `references/zustand.md` |
| Redux Toolkit | `references/redux-toolkit.md` |
| TanStack Query | `references/tanstack-query.md` |
| Ant Design v5 | `references/antd.md` |
| TanStack Table | `references/tanstack-table.md` |
| ESLint + Prettier | `references/eslint-prettier.md` |
| Git Hooks (Husky) | `references/git-hooks.md` |
| Testing | `references/testing.md` |
| API Service Layer | `references/api-services.md` |
| Environments | `references/environments.md` |

---

## Step 3: Generate Project Output

Produce a **complete, runnable** project scaffold. Generate files in order below.

---

### 3A — `package.json`

Include ALL chosen deps. Scripts section must include:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "build:stage": "dotenv -e .env.stage -- next build",
    "build:prod": "dotenv -e .env.production -- next build",
    "start:local": "dotenv -e .env.local -- next dev",
    "start:stage": "dotenv -e .env.stage -- next start",
    "start:prod": "dotenv -e .env.production -- next start",
    "lint": "next lint",
    "lint:fix": "eslint --fix .",
    "format": "prettier --write .",
    "type-check": "tsc --noEmit",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest run --coverage",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:report": "playwright show-report",
    "prepare": "husky"
  }
}
```

For Vite projects, replace `next dev/build/start` with `vite`, `vite build`, `vite preview`.

---

### 3B — Environment Files

Generate **4 files** — see `references/environments.md` for full content.

```
.env.local          # local dev — never committed
.env.stage          # staging — committed (no secrets)
.env.production     # production — committed (no secrets, use CI for secrets)
.env.example        # template showing all keys — always committed
```

Standard env vars to include:
```
NEXT_PUBLIC_API_URL=
NEXT_PUBLIC_APP_ENV=local|stage|production
NEXT_PUBLIC_APP_NAME=
API_SECRET_KEY=           # server-only (no NEXT_PUBLIC_ prefix)
NEXT_PUBLIC_SENTRY_DSN=
```

---

### 3C — TypeScript Config

Always strict:

```json
{
  "compilerOptions": {
    "strict": true,
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "baseUrl": ".",
    "paths": { "@/*": ["./src/*"] },
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "forceConsistentCasingInFileNames": true,
    "incremental": true
  }
}
```

---

### 3D — ESLint + Prettier

Use **ESLint v9 flat config** (`eslint.config.mjs`). See `references/eslint-prettier.md`.

Key rules to always include:
- `@typescript-eslint/no-explicit-any` — warn
- `react-hooks/exhaustive-deps` — error
- `import/order` with groups: builtin → external → internal → relative
- `no-console` — warn (allow warn/error)

`.prettierrc`:
```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100,
  "bracketSpacing": true
}
```

---

### 3E — Git Hooks (Husky)

See `references/git-hooks.md` for full setup. Generate:

**`.husky/pre-commit`** — runs lint-staged:
```sh
#!/usr/bin/env sh
npx lint-staged
```

**`.husky/pre-push`** — runs type-check + unit tests:
```sh
#!/usr/bin/env sh
echo "Running pre-push checks..."
npm run type-check && npm run test
```

**`lint-staged` config in `package.json`**:
```json
{
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{js,jsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md,yml,yaml}": ["prettier --write"]
  }
}
```

---

### 3F — API Service Layer

See `references/api-services.md`. Generate these files:

```
src/lib/
├── api/
│   ├── client.ts          # Axios instance with interceptors
│   ├── endpoints.ts       # All API URL constants
│   └── types.ts           # Shared API response types
src/services/
│   ├── userService.ts     # Example: user CRUD operations
│   └── index.ts           # Re-exports all services
src/hooks/api/
│   ├── useUsers.ts        # TanStack Query hooks wrapping userService
│   └── index.ts
```

`src/lib/api/client.ts` template:
```typescript
import axios from 'axios';

const apiClient = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  timeout: 10000,
  headers: { 'Content-Type': 'application/json' },
});

// Request interceptor — attach auth token
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

// Response interceptor — global error handling
apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // handle unauthorized
    }
    return Promise.reject(error);
  }
);

export default apiClient;
```

---

### 3G — TanStack Query Setup

Generate `src/lib/queryClient.ts` and wrap app root with `QueryClientProvider`.

```typescript
// src/lib/queryClient.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,  // 5 minutes
      retry: 1,
      refetchOnWindowFocus: false,
    },
  },
});
```

For Next.js App Router, wrap in `src/app/providers.tsx`:
```tsx
'use client';
import { QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { queryClient } from '@/lib/queryClient';

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

---

### 3H — Testing Setup

See `references/testing.md` for complete configs.

**Vitest** (`vitest.config.ts`):
```typescript
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./src/test/setup.ts'],
    coverage: {
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/', 'src/test/'],
    },
  },
  resolve: { alias: { '@': path.resolve(__dirname, './src') } },
});
```

**Playwright** (`playwright.config.ts`):
```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  reporter: 'html',
  use: {
    baseURL: process.env.PLAYWRIGHT_BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'Mobile Chrome', use: { ...devices['Pixel 5'] } },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

Generate starter test files:
- `src/test/setup.ts` — testing-library setup
- `src/components/ui/Button.test.tsx` — sample unit test
- `e2e/home.spec.ts` — sample Playwright e2e test

---

### 3I — Folder Structure

```
<project-name>/
├── .husky/
│   ├── pre-commit
│   └── pre-push
├── e2e/
│   └── home.spec.ts
├── public/
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   └── providers.tsx
│   ├── components/
│   │   ├── ui/                 # Generic: Button, Modal, Input
│   │   └── layout/             # Header, Sidebar, Footer
│   ├── features/               # Feature modules
│   │   └── example/
│   │       ├── components/
│   │       ├── hooks/
│   │       └── store.ts
│   ├── hooks/
│   │   └── api/                # TanStack Query hooks
│   ├── lib/
│   │   ├── api/
│   │   │   ├── client.ts
│   │   │   ├── endpoints.ts
│   │   │   └── types.ts
│   │   └── queryClient.ts
│   ├── services/               # API service functions
│   │   ├── userService.ts
│   │   └── index.ts
│   ├── store/                  # Zustand or Redux store
│   ├── styles/
│   │   ├── _variables.scss
│   │   ├── _mixins.scss
│   │   └── globals.scss
│   ├── test/
│   │   └── setup.ts
│   ├── types/                  # Shared TS types
│   └── constants/
├── .env.local
├── .env.stage
├── .env.production
├── .env.example
├── .eslintignore
├── .gitignore
├── .prettierrc
├── CLAUDE.md               # or AGENTS.md — per user choice in Q4
├── eslint.config.mjs
├── next.config.ts
├── package.json
├── playwright.config.ts
├── tsconfig.json
└── vitest.config.ts
```

---

### 3J — `.gitignore`

Always include:
```
node_modules/
.next/
dist/
out/
.env.local
*.local
.DS_Store
coverage/
playwright-report/
test-results/
```

---

### 3K — AI Rules File (`CLAUDE.md` or `AGENTS.md`)

Generate the file chosen in **Q4** at the project root. Content is identical regardless of file name:

```markdown
# ================================
# AI PROJECT RULES
# ================================

## 1. CORE PRINCIPLES
- Treat rules as persistent system behavior
- TypeScript only — never output `.js` or `.jsx`
- Act as a senior dev: skip obvious explanations
- Prioritize clarity, minimal tokens, direct execution

## 2. NAMING CONVENTIONS
- **Files:** `kebab-case` → `user-profile.tsx`, `auth-service.ts`
- **Folders:** `kebab-case` → `components/`, `api-services/`
- **Components:** `PascalCase` → `UserProfile`, `AuthButton`
- **Functions:** `camelCase` → `getUserById`, `handleSubmit`
- **Variables:** `camelCase` → `isLoading`, `userData`
- **Constants:** `SCREAMING_SNAKE_CASE` → `MAX_RETRY_COUNT`
- **Types/Interfaces:** `PascalCase` → `UserProfile`, `ApiResponse`
- **Generics:** descriptive → `TData`, `TResponse` (not `T` unless conventional)
- **Booleans:** prefix `is/has/can/should` → `isActive`, `hasPermission`
- **Event handlers:** prefix `handle` → `handleClick`, `handleFormSubmit`
- **Hooks:** prefix `use` → `useAuth`, `useUserData`

## 3. FILE HANDLING
- NEVER guess file/folder paths — ask if missing
- Read only the specific file needed
- Never load full directories

## 4. FILE CREATION
- All files: `.ts` or `.tsx` only
- Follow project-structure.md for placement
- Reuse existing utilities/hooks/components before creating new
- Mirror naming conventions already in project
- One component per file

## 5. STRUCTURE MANAGEMENT
- Do NOT auto-update project-structure.md
- Suggest update only if: new folder type introduced OR architecture changes
- Ask before modifying — one suggestion per conversation

## 6. EXECUTION PRIORITY
1. User instruction
2. Explicit file path
3. Existing codebase patterns
4. project-structure.md
5. General best practices

## 7. TOKEN OPTIMIZATION
- No filler phrases, no closing summaries unless asked
- No repeating user's words back
- Omit comments unless logic is non-obvious
- Show only changed block for edits — not full file
- Small fixes (<5 lines): inline, no explanation
- Large changes: 2–3 line plan, then code

## 8. CODE QUALITY
- Early returns to reduce nesting
- `const` over `let`, never `var`
- Destructure props and imports
- Optional chaining `?.` and nullish coalescing `??`
- Functions under 30 lines — split if longer
- Named constants — no magic numbers
- No commented-out code
- `async/await` over `.then()`
- Error handling at boundary (`try/catch` at top level)
- Never hardcode secrets or credentials

## 9. TYPESCRIPT
- No `any` — use `unknown` + narrow, or define proper type
- No `@ts-ignore` unless unavoidable — comment why if used
- All params and return types explicitly typed
- `interface` for object shapes, `type` for unions/intersections
- `as const` for fixed literal objects/arrays
- `satisfies` to validate without losing literal inference
- No `as SomeType` assertions — fix the root type
- Shared types → `types/` or `*.types.ts`
- Never use `Function` type — define specific signature
- Use `Readonly<T>`, `NonNullable<T>`, `ReturnType<T>` utility types

## 10. ESLINT RULES
- All output code must be ESLint-clean — zero warnings, zero errors
- Follow rule severity: `error` blocks output, `warn` must be resolved
- **Line rules:**
  - Max line length: **100 chars** — break longer lines
  - Max file length: **300 lines** — split if exceeded
  - No trailing whitespace
  - Always a newline at end of file
- **Type rules:**
  - `@typescript-eslint/no-explicit-any` — enforced, no exceptions
  - `@typescript-eslint/explicit-function-return-type` — all functions typed
  - `@typescript-eslint/no-unused-vars` — remove before output
  - `@typescript-eslint/consistent-type-imports` — use `import type`
  - `@typescript-eslint/no-non-null-assertion` — no `!` assertions
- **Logic rules:**
  - `no-console` — use a logger utility, never raw `console.log`
  - `eqeqeq` — always `===`, never `==`
  - `no-shadow` — no variable shadowing outer scope
  - `prefer-const` — enforced
  - `no-var` — enforced
- **React rules (if applicable):**
  - `react-hooks/rules-of-hooks` — hooks only at top level
  - `react-hooks/exhaustive-deps` — all deps in dependency arrays
  - `react/self-closing-comp` — `<Component />` not `<Component></Component>`

## 11. PRETTIER FORMATTING
- All output code must be Prettier-formatted before responding
- **Config (match project `.prettierrc` if present, else use these defaults):**
```json
  {
    "semi": true,
    "singleQuote": true,
    "trailingComma": "all",
    "printWidth": 100,
    "tabWidth": 2,
    "useTabs": false,
    "bracketSpacing": true,
    "arrowParens": "always",
    "endOfLine": "lf"
  }
```
- Single quotes for strings — `'value'` not `"value"`
- Always trailing commas in multiline (params, arrays, objects)
- Arrow functions always wrap params — `(x) => x` not `x => x`
- Opening brace on same line — no Allman style
- Multiline ternaries: condition on its own line
- Object destructuring: one prop per line if 3+ props

## 12. IMPORTS
- Check existing utils/hooks before adding a new dependency
- Stick to project's existing UI library — never mix
- Group order (enforced by `eslint-plugin-import`):
  1. External libs
  2. Internal modules (absolute paths)
  3. Relative imports
  4. Type imports (`import type`)
  5. Styles
- Blank line between each group
- Remove unused imports before output
- `import type { }` for type-only imports

## 13. COMPONENTS
- Props interface defined above component
- No logic inside JSX — extract to variables/handlers
- JSX under 50 lines — extract sub-components if longer
- No inline styles unless truly one-off
- Semantic HTML: `button`, `nav`, `main`, `section`

## 14. STATE & DATA
- Keep state as local as possible — lift only when needed
- Derive values from state — no redundant state
- API calls in service files, not components
- Always handle loading/error/success explicitly

## 15. SAFETY CHECK (before every task)
- File path clear? → No → ask
- Creating a duplicate? → Yes → flag it
- New library needed? → Yes → confirm with user first
- Output is `.ts`/`.tsx`? → No → fix before responding
- New pattern introduced? → Yes → suggest structure update
- Code passes ESLint? → No → fix before responding
- Code is Prettier-formatted? → No → format before responding
```

> **Default:** If the user did not answer Q4 (e.g. bypass mode), generate `CLAUDE.md`.

---

## Step 4: Deliver Files

1. Use `create_file` to write every file to `/home/claude/<project-name>/`
2. Run `zip -r <project-name>.zip <project-name>/` via bash
3. Use `present_files` to share the ZIP and key config files

End with a **Getting Started** section:

```bash
# Install dependencies
npm install   # or: pnpm install | yarn

# Set up Git hooks
npm run prepare

# Copy environment template
cp .env.example .env.local
# (edit .env.local with your values)

# Start local dev
npm run start:local

# Run unit tests
npm test

# Run E2E tests (requires dev server running)
npm run test:e2e

# Build for staging
npm run build:stage

# Build for production
npm run build:prod
```

---

## Quality Rules

- **Never use CRA (Create React App)** — deprecated. Always Vite or Next.js.
- **Always TypeScript strict mode** unless user explicitly says JS.
- **ESLint flat config** for all new projects (Node 18+).
- **Tailwind v4**: uses CSS-based config. Tailwind v3: uses `tailwind.config.ts`. Ask if unclear.
- **antd v5**: no need for `import 'antd/dist/reset.css'`. Uses CSS-in-JS via `StyleProvider`.
- When **Tailwind + antd**: add `corePlugins: { preflight: false }` to prevent CSS conflicts.
- **TanStack Table v8 is headless** — pair with antd or Tailwind table styles.
- **Never put business logic in page components** — use `features/` or `hooks/`.
- **`.env.local` is never committed** — always in `.gitignore`.
- **Pre-push hook** must run `type-check` AND `test` — not just lint.
- **Axios client** must have interceptors for auth token and 401 handling from the start.
- **All API calls** go through `src/services/` — never call `apiClient` directly from components.

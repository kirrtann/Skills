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
  scaffold a full Next.js 15 App Router project with the default stack immediately.
  Generates complete file trees, config files, scripts, and boilerplate code tailored to the
  user's chosen stack and version.
---

# Frontend Project Setup Skill

Scaffold complete, production-ready frontend projects. Always generate **real files** with **version-correct configs** — not just instructions.

---

## ⚡ BYPASS / DEFAULT MODE

If user says **"bypass"**, **"default"**, **"skip"**, or **"just set it up"** — skip all questions and scaffold immediately with:

| Option | Default |
|---|---|
| Framework | **Next.js 15 App Router** |
| Styling | **Tailwind CSS v4 + SCSS** |
| State | **Zustand + TanStack Query v5** |
| UI Library | **Ant Design v5** |
| Tables | **TanStack Table v8** |
| Icons | **React Icons + Ant Design Icons** |
| Language | **TypeScript 5 (strict)** |
| Linting | **ESLint v9 flat config + Prettier** |
| Testing | **Playwright + Vitest** |
| Git Hooks | **Husky + lint-staged** |
| API Layer | **Axios + TanStack Query hooks** |
| Environments | **local / stage / production `.env` files** |

Jump directly to **Step 3** with these defaults.

---

## Step 1: Gather Stack Choices

Use `ask_user_input_v0`. Max 3 questions per round.

### Q1 — Framework & Version
- **Framework**: Next.js 15 ★ | Next.js 14 | React + Vite
- **Router** (Next.js only): App Router ★ | Pages Router

### Q2 — Styling & UI
- **Styling**: Tailwind v4 ★ | Tailwind v3 | SCSS | Tailwind + SCSS ★ | CSS Modules
- **UI Library**: Ant Design v5 ★ | shadcn/ui | MUI v6 | Chakra UI v3 | None

### Q3 — State, Testing & Tooling
- **State**: Zustand ★ | Redux Toolkit | Context only | None
- **Data Fetching**: TanStack Query v5 ★ | SWR | RTK Query | None
- **Testing**: Playwright + Vitest ★ | Playwright only | Vitest only | None
- **Git Hooks**: Husky ★ | Lefthook | None

### Q4 — AI Rules File
- `CLAUDE.md` ★ (Claude / Anthropic tooling) | `AGENTS.md` (OpenAI Codex)

★ = recommended default

---

## Step 2: Select Reference Files

Read **all relevant** reference files before generating:

| Topic | File |
|---|---|
| Next.js App Router | `references/nextjs-app.md` |
| React + Vite | `references/vite-react.md` |
| Tailwind CSS | `references/tailwind.md` |
| SCSS | `references/scss.md` |
| Zustand | `references/zustand.md` |
| Redux Toolkit | `references/redux-toolkit.md` |
| TanStack Query | `references/tanstack-query.md` |
| Ant Design v5 | `references/antd.md` |
| TanStack Table | `references/tanstack-table.md` |
| ESLint + Prettier | `references/eslint-prettier.md` |
| Git Hooks | `references/git-hooks.md` |
| Testing | `references/testing.md` |
| API Service Layer | `references/api-services.md` |
| Environments | `references/environments.md` |

---

## Step 3: VERSION-AWARE CONFIG GENERATION

> ⚠️ This is the core of the skill. Config file format, syntax, and filenames differ by version.
> Always emit the correct variant based on the user's chosen versions.

---

### 3A — Framework Init Command

#### Next.js 15 (App Router)
```bash
npx create-next-app@latest my-app \
  --typescript --tailwind --eslint \
  --app --src-dir --import-alias "@/*" \
  --turbopack
```

#### Next.js 14 (App Router)
```bash
npx create-next-app@latest my-app \
  --typescript --tailwind --eslint \
  --app --src-dir --import-alias "@/*"
```

#### Next.js 14/15 (Pages Router)
Same flags but omit `--app`, add `--no-app`.

#### Vite + React
```bash
npm create vite@latest my-app -- --template react-ts
```

---

### 3B — Next.js Config (VERSION-SENSITIVE)

> 🚨 **MANDATORY VERSION CHECK — do this before writing any next.config file:**
>
> 1. If `package.json` is available, read the `next` version from it.
> 2. If scaffolding fresh, use the version the user stated in Q1.
> 3. Apply the rule below — **wrong file extension = instant runtime crash.**
>
> | Next.js version | Config filename | Format |
> |---|---|---|
> | **15.x and above** | `next.config.ts` | TypeScript — `import type { NextConfig }` |
> | **14.x and below** | `next.config.mjs` | ESM JS — `/** @type */` JSDoc only |
>
> ❌ `next.config.ts` on Next.js 14 throws: *"Configuring Next.js via 'next.config.ts' is not supported"*
> ❌ `next.config.js` (CJS) on either version is legacy — never emit it for new projects

#### Next.js 15 → `next.config.ts`
```ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  reactStrictMode: true,
  images: {
    remotePatterns: [
      { protocol: 'https', hostname: '**.example.com' },
    ],
  },
}

export default nextConfig
```

#### Next.js 14 → `next.config.mjs`
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  images: {
    remotePatterns: [
      { protocol: 'https', hostname: '**.example.com' },
    ],
  },
}

export default nextConfig
```

---

### 3C — Tailwind Config (VERSION-SENSITIVE)

#### Tailwind v4 → CSS-only config (NO `tailwind.config.ts`)

```css
/* src/app/globals.css  (Next.js)  OR  src/styles/global.css  (Vite) */
@import "tailwindcss";

@theme {
  --color-brand-50:  #eff6ff;
  --color-brand-500: #3b82f6;
  --color-brand-900: #1e3a8a;
  --font-sans: 'Inter', sans-serif;
  --breakpoint-sm: 640px;
  --breakpoint-md: 768px;
  --breakpoint-lg: 1024px;
  --breakpoint-xl: 1280px;
}
```

Install:
```bash
npm install -D tailwindcss@next @tailwindcss/vite   # Vite
npm install -D tailwindcss@next                      # Next.js (built-in support)
```

> **No `tailwind.config.ts` file for v4.** Do NOT generate one.

#### Tailwind v3 → `tailwind.config.ts` (TypeScript file required)

```ts
import type { Config } from 'tailwindcss'

const config: Config = {
  content: [
    './src/**/*.{js,ts,jsx,tsx,mdx}',
    './src/app/**/*.{js,ts,jsx,tsx,mdx}',
    './src/components/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        brand: { 50: '#eff6ff', 500: '#3b82f6', 900: '#1e3a8a' },
      },
      fontFamily: { sans: ['Inter', 'sans-serif'] },
    },
  },
  plugins: [],
  // When combining with Ant Design:
  // corePlugins: { preflight: false },
}

export default config
```

CSS import (v3):
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Install (v3):
```bash
npm install -D tailwindcss@3 postcss autoprefixer
npx tailwindcss init -p
```

---

### 3D — Vite Config (VERSION-SENSITIVE)

#### Vite 5 + Tailwind v4
```ts
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'
import path from 'path'

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: { alias: { '@': path.resolve(__dirname, './src') } },
})
```

#### Vite 5 + Tailwind v3
```ts
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  resolve: { alias: { '@': path.resolve(__dirname, './src') } },
})
// postcss.config.js handles Tailwind v3 separately
```

#### postcss.config.js (Tailwind v3 only — omit for v4)
```js
export default {
  plugins: { tailwindcss: {}, autoprefixer: {} },
}
```

---

### 3E — ESLint Config (VERSION-SENSITIVE)

#### ESLint v9 → `eslint.config.mjs` (flat config — default for new projects)

```js
import js from '@eslint/js'
import tsPlugin from '@typescript-eslint/eslint-plugin'
import tsParser from '@typescript-eslint/parser'
import reactPlugin from 'eslint-plugin-react'
import reactHooks from 'eslint-plugin-react-hooks'
import jsxA11y from 'eslint-plugin-jsx-a11y'
import importPlugin from 'eslint-plugin-import'
import prettierConfig from 'eslint-config-prettier'

export default [
  js.configs.recommended,
  {
    files: ['**/*.{ts,tsx}'],
    languageOptions: {
      parser: tsParser,
      parserOptions: { project: './tsconfig.json', ecmaFeatures: { jsx: true } },
    },
    plugins: {
      '@typescript-eslint': tsPlugin,
      react: reactPlugin,
      'react-hooks': reactHooks,
      'jsx-a11y': jsxA11y,
      import: importPlugin,
    },
    rules: {
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      '@typescript-eslint/no-explicit-any': 'warn',
      '@typescript-eslint/consistent-type-imports': 'error',
      'react/react-in-jsx-scope': 'off',
      'react/prop-types': 'off',
      'react/self-closing-comp': 'error',
      'react-hooks/rules-of-hooks': 'error',
      'react-hooks/exhaustive-deps': 'warn',
      'import/order': ['error', {
        groups: ['builtin', 'external', 'internal', 'parent', 'sibling', 'index'],
        'newlines-between': 'always',
        alphabetize: { order: 'asc' },
      }],
      'no-console': ['warn', { allow: ['warn', 'error'] }],
      'prefer-const': 'error',
      'no-var': 'error',
    },
    settings: { react: { version: 'detect' } },
  },
  prettierConfig, // always last
]
```

#### ESLint v8 → `.eslintrc.cjs` (legacy — only if user targets Node < 18 or existing project)

```js
module.exports = {
  root: true,
  env: { browser: true, es2020: true },
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'plugin:jsx-a11y/recommended',
    'prettier',
  ],
  parser: '@typescript-eslint/parser',
  rules: {
    'react/react-in-jsx-scope': 'off',
    '@typescript-eslint/no-explicit-any': 'warn',
  },
}
```

> **Default to ESLint v9 flat config** for all new projects. Only use v8 if explicitly requested.

---

### 3F — TypeScript Config

Always strict. Use `moduleResolution: "bundler"` for Next.js 14+/Vite 4+:

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
    "incremental": true,
    "skipLibCheck": true
  },
  "include": ["src", "next.config.ts"],
  "exclude": ["node_modules"]
}
```

---

### 3G — package.json

> ⚠️ **CRITICAL — Framework-Specific Dependencies:**
>
> - **Next.js:** Do NOT include `@tailwindcss/vite` — it breaks Next.js builds. Include only `tailwindcss@next`.
> - **Vite:** Include both `tailwindcss@next` AND `@tailwindcss/vite` for CSS-first config.
> - **@types/node:** Must be `^20.19.x` or higher for Next.js 15 with Tailwind v4 (peer dependency fix).
> - **@types/react:** Match React version — `^19.x` for React 19, `^18.x` for React 18.

Include ALL chosen deps with correct versions. Always pin major versions in descriptions.

#### Scripts
```json
{
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "next build",
    "start": "next start",
    "build:stage": "dotenv -e .env.stage -- next build",
    "build:prod": "dotenv -e .env.production -- next build",
    "start:local": "dotenv -e .env.local -- next dev --turbopack",
    "start:stage": "dotenv -e .env.stage -- next start",
    "start:prod": "dotenv -e .env.production -- next start",
    "lint": "next lint",
    "lint:fix": "eslint --fix .",
    "format": "prettier --write .",
    "type-check": "tsc --noEmit",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "prepare": "husky"
  }
}
```

For Vite: replace `next dev/build/start` with `vite`, `vite build`, `vite preview`. Remove `--turbopack`.
For Next.js 14: remove `--turbopack` from dev script.

#### Key version table (emit correct versions in dependencies block):

| Package | Latest stable | Notes |
|---|---|---|
| **next** | **15.x** | Use 14.x if user chose v14 |
| **react / react-dom** | **19.x (Next.js 15) / 18.x (Next.js 14)** | Must match |
| **typescript** | **^5.5.x** | |
| **@types/node** | **^20.19.x** | ⚠️ Must be ≥20.19.0 for Tailwind v4 compatibility |
| **@types/react** | **^19.x** | Match React major version |
| tailwindcss | ^4.x or ^3.x | Match chosen version |
| @tailwindcss/vite | ^4.x | ⚠️ **Vite ONLY** — DO NOT include for Next.js |
| @tanstack/react-query | ^5.x | v5 has breaking changes from v4 |
| @tanstack/react-table | ^8.x | |
| zustand | ^5.x | v5 released 2024 |
| antd | ^5.x | |
| @ant-design/icons | ^5.x | |
| axios | ^1.7.x | |
| eslint | ^9.x | Flat config |
| husky | ^9.x | v9 changed init command |
| lint-staged | ^15.x | |
| vitest | ^2.x | |
| @playwright/test | ^1.47.x | |

---

### 3H — Environment Files

Generate 4 files — see `references/environments.md`:

```
.env.local        ← never committed
.env.stage        ← committed (no secrets)
.env.production   ← committed (no secrets)
.env.example      ← always committed (all keys, no values)
```

For Next.js: prefix client-side vars with `NEXT_PUBLIC_`.
For Vite: prefix client-side vars with `VITE_` and use `import.meta.env` in code.

---

### 3I — Git Hooks

See `references/git-hooks.md`. Husky v9 changed init:

```bash
# Husky v9 (current) — init command
npx husky init
# This creates .husky/pre-commit automatically
```

`.husky/pre-commit`:
```sh
npx lint-staged
```

`.husky/pre-push`:
```sh
echo "Running pre-push checks..."
npm run type-check && npm run test
```

`package.json` lint-staged config:
```json
{
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{js,jsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md,yml,yaml,css,scss}": ["prettier --write"]
  }
}
```

---

### 3J — Project Structure

#### Next.js 15 App Router
```
src/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── globals.css
│   └── (dashboard)/
│       └── dashboard/
│           ├── layout.tsx
│           └── page.tsx
├── components/
│   └── ui/           ← shared primitives
├── features/         ← domain-scoped logic
├── hooks/
├── lib/
│   └── api/
│       ├── client.ts
│       ├── endpoints.ts
│       └── types.ts
├── services/
├── store/            ← Zustand or Redux
├── styles/
└── types/
```

#### Vite + React
```
src/
├── main.tsx
├── App.tsx
├── routes/
├── components/ui/
├── features/
├── hooks/
├── lib/api/
├── services/
├── store/
├── styles/
└── types/
```

---

### 3K — Core Boilerplate Files

#### `src/lib/utils.ts` (always emit this)
```ts
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

#### `src/lib/api/client.ts` — see `references/api-services.md`

#### Providers wrapper — Next.js App Router
```tsx
// src/app/providers.tsx
'use client'

import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'
import { ConfigProvider, App as AntApp } from 'antd'
import { StyleProvider } from '@ant-design/cssinjs'
import { useState } from 'react'

const theme = {
  token: {
    colorPrimary: '#3b82f6',
    borderRadius: 8,
    fontFamily: 'Inter, sans-serif',
  },
}

export function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(
    () => new QueryClient({
      defaultOptions: {
        queries: { staleTime: 60 * 1000, retry: 1, refetchOnWindowFocus: false },
      },
    })
  )

  return (
    <QueryClientProvider client={queryClient}>
      <StyleProvider hashPriority="high">
        <ConfigProvider theme={theme}>
          <AntApp>{children}</AntApp>
        </ConfigProvider>
      </StyleProvider>
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  )
}
```

```tsx
// src/app/layout.tsx
import type { Metadata } from 'next'
import { Inter } from 'next/font/google'
import { Providers } from './providers'
import './globals.css'

const inter = Inter({ subsets: ['latin'] })

export const metadata: Metadata = { title: 'My App', description: 'Production app' }

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <Providers>{children}</Providers>
      </body>
    </html>
  )
}
```

---

### 3L — .gitignore

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
.turbo/
```

---

### 3M — AI Rules File (`CLAUDE.md` or `AGENTS.md`)

Place at project root. Content is identical regardless of file name:

```markdown
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

---

## Step 4: Deliver Files

1. `create_file` every file to `/home/claude/<project-name>/`
2. Run `zip -r <project-name>.zip <project-name>/`
3. `present_files` to share the ZIP

End with Getting Started:

```bash
npm install
npm run prepare         # initialize Git hooks
cp .env.example .env.local  # fill in your values
npm run start:local
npm test
npm run test:e2e
npm run build:prod
```

---

## Quality Rules

> 🚨 **MANDATORY CRITICAL CHECKS — FAILURE = BROKEN PROJECT:**

1. **Next.js version in package.json = Next.js config file type:**
   - ✅ Next.js 15+: `next.config.ts` (TypeScript)
   - ✅ Next.js 14 and below: `next.config.mjs` (ESM JS + JSDoc)
   - ❌ NEVER `next.config.js` (CJS) for new projects
   - ❌ NEVER `next.config.ts` on Next.js 14 — instant crash

2. **Tailwind v4 + @types/node compatibility:**
   - ✅ Use `@types/node@^20.19.x` or higher for Tailwind v4
   - ✅ Tailwind v4: NO `tailwind.config.ts` — CSS-only via `@import "tailwindcss"`
   - ❌ NEVER `@tailwindcss/vite` for Next.js projects (Vite only)
   - ❌ NEVER old @types/node versions (causes peer dependency conflicts)

3. **Framework-specific dependencies:**
   - ✅ **Next.js 15:** `tailwindcss@next` only (no Vite plugin)
   - ✅ **Vite:** `tailwindcss@next` + `@tailwindcss/vite`
   - ✅ **@types/react:** Match React version (^19.x for React 19, ^18.x for React 18)

4. **TypeScript config must include:**
   - `"moduleResolution": "bundler"` for Next.js 14+
   - `"strict": true` — always
   - Correct include paths for your framework

---

## Legacy Rules (kept for reference)

- **Never CRA** — always Vite or Next.js
- **Always TypeScript strict** unless user says JS
- **Next.js 15+**: config → `next.config.ts` (TypeScript) ✅
- **Next.js 14 and below**: config → `next.config.mjs` (ESM JS, JSDoc only) ✅
- **ALWAYS** confirm Next.js version from `package.json` or user input before writing config
- **NEVER** emit `next.config.ts` unless version is confirmed ≥ 15 — it crashes on 14
- **NEVER** emit `next.config.js` (CJS) for new projects
- **Tailwind v4**: CSS-only config via `@import "tailwindcss"` — NO `tailwind.config.ts`
- **Tailwind v3**: `tailwind.config.ts` required + `postcss.config.js`
- **ESLint v9**: `eslint.config.mjs` flat config
- **ESLint v8**: `.eslintrc.cjs` (legacy only)
- **Husky v9**: `npx husky init` (not `npx husky install`)
- **TanStack Query v5**: `gcTime` not `cacheTime`, `initialPageParam` required in infinite queries
- **Zustand v5**: no breaking changes from v4 API — same patterns apply
- **antd v5 + Tailwind**: always add `corePlugins: { preflight: false }` in Tailwind v3, or handle via CSS specificity in v4
- **`.env.local` never committed** — always in `.gitignore`
- **Pre-push** must run `type-check` AND `test`
- **Axios client** must have auth + 401 interceptors from the start
- **All API calls** go through `src/services/` only


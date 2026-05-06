---
name: setup
description: Scaffold a new Next.js project with the full multi-agent team toolchain. Creates package.json, GitHub Actions CI, Lighthouse CI config, ESLint, Prettier, Husky, and Vercel config.
---

Scaffold a new Next.js project: $ARGUMENTS

@devops Set up a complete Next.js project with this configuration:
- Project name: $ARGUMENTS (or "my-app" if not specified)
- Stack: Next.js 14+, TypeScript strict, Tailwind CSS, next-intl
- State: Zustand + React Query
- Testing: Playwright + Vitest + MSW
- CI: GitHub Actions (type-check → lint → unit → E2E → Lighthouse)
- Deploy target: Vercel

Produce ALL config files needed:
1. `package.json` with all dependencies and scripts
2. `.github/workflows/ci.yml`
3. `lighthouserc.mjs` with performance budgets
4. `.env.example`
5. `vercel.json` with security headers
6. `eslint.config.mjs`
7. `.prettierrc`
8. `commitlint.config.mjs`
9. `.husky/pre-commit` and `.husky/commit-msg`
10. `tsconfig.json` with strict settings

Then output the exact setup commands to run in order.

# 🚀 AI Development Agent System

Automated multi-agent framework for software development workflows.

---

## 📋 Overview

| Component | Purpose |
|-----------|---------|
| **Agents** (8) | Specialized AI experts |
| **Skills** (10+) | Reusable modules |
| **Commands** (6) | Orchestrated pipelines |

---

## 📁 Structure

```
agents/          # admin, react-dev, ui-architect, testing, code-quality, perf, devops, i18n-a11y
skills/          # frontend-project-setup, playwright-testing, i18n-translation, css-refactor, etc.
commands/        # setup, build-feature, review, audit-a11y, audit-perf, agent-status
AGENTS.md        # Global rules & conventions
```

---

## 🤖 Agents

| Agent | Does |
|-------|------|
| **admin** | Task decomposition → plans |
| **react-dev** | React component implementation |
| **ui-architect** | UI design & specs |
| **testing** | E2E (Playwright) & unit tests |
| **code-quality** | Code review & scoring |
| **perf** | Core Web Vitals optimization |
| **devops** | CI/CD & deployment |
| **i18n-a11y** | Translations & WCAG compliance |

---

## 🛠️ Skills

| Skill | Purpose |
|-------|---------|
| **frontend-project-setup** | Scaffold React/Next.js/Vite projects |
| **playwright-testing** | E2E test automation |
| **i18n-translation** | Multi-language support |
| **git-commit-changelog** | Auto-generate changelogs |
| **css-refactor** | Tailwind/CSS optimization |
| **lighthouse-audit** | Performance & SEO audit |
| **component-dependency-visualizer** | Visualize dependencies |
| **frontend-security-scanner** | Security audit |
| **font** | Font optimization |

---

## ⚡ Commands

| Command | Purpose |
|---------|---------|
| **build-feature** | Full pipeline: decompose → design → build → test → review |
| **setup** | New project initialization |
| **review** | Code quality review |
| **audit-a11y** | Accessibility check |
| **audit-perf** | Performance analysis |
| **agent-status** | System health check |

---

## 🚀 Quick Start

**New Project:**
```
Invoke: @setup
Output: Production-ready project
```

**Build Feature:**
```
Invoke: @build-feature
Input: Feature description
Output: Implemented + tested feature
```

**Audit:**
```
Invoke: @audit-a11y or @audit-perf
Output: Compliance & optimization report
```

---

## 📍 .claude vs .agents Simple

Use `.claude/` for Claude-only • Use `.agents/` for framework-agnostic (current)

---

## 🏗️ Tech Stack

React 19 • Next.js 15 • TypeScript • Tailwind + SCSS • Zustand + React Query • Vitest + Playwright • ESLint + Prettier

---

## 🔑 Core Rules

- TypeScript only (`.ts`/`.tsx`)
- No `any` — proper types only
- Max 100 char lines, 300 line files  
- kebab-case files, PascalCase components, camelCase functions
- Single quotes `'`, early returns, reduce nesting
- React Query for async, Zustand for state
- ESLint + Prettier compliance

---

## 🎯 When to Use

- **Agents:** Domain expertise & complex decisions
- **Skills:** Specific focused tasks
- **Commands:** Multi-agent orchestration

---

## 📖 Naming

✅ `agents/react-dev.md` • ✅ `skills/frontend-project-setup/SKILL.md`

---

## 🔗 Key Files

- [AGENTS.md](AGENTS.md) — Rules & conventions
- [agents/](agents/) — Agent definitions
- [skills/](skills/) — Skill modules
- [commands/](commands/) — Command pipelines

---

**Agents:** 8 | **Skills:** 10+ | **Commands:** 6 | **Status:** Active ✅
---
name: component-dependency-visualizer
description: >
  Deep-analyze React/Next.js projects for component architecture issues, dependency coupling,
  re-render problems, state flow anti-patterns, bundle bloat, performance issues, and structural
  vulnerabilities — then render an animated interactive dependency graph artifact in chat.
  Use this skill whenever the user wants to visualize, audit, or understand their React/Next.js
  component structure or performance — even if they don't say "visualize". Triggers include:
  "analyze my components", "show component tree", "find re-render issues", "state flow problems",
  "why is my app slow", "bundle size issues", "dependency graph", "component architecture review",
  "performance audit", "too many re-renders", "prop drilling", "analyze my project", "what's wrong
  with my React app", "component coupling", "check my Next.js structure", "show component dependencies",
  "visualize my project", "audit my codebase", "find circular imports", "lazy loading issues",
  or any request to understand, visualize, or improve React/Next.js component architecture.
  Always output both a written report in chat AND an animated interactive React artifact graph.
---

# Component Dependency Visualizer

Senior-level React/Next.js architecture analysis. Produces a written audit report in chat
AND a live animated interactive dependency graph rendered as an artifact.

---

## Core Rules

1. **Always generate both outputs** — written report in chat + animated artifact (mandatory)
2. **Report in chat only** — never create files unless user explicitly asks
3. **Never auto-fix** — always ask which issues to fix after the report
4. **Scan the whole project first** — build a full picture before writing anything

---

## Step 1 — Project Scan

Run these bash commands:

```bash
# Map all components and pages
find . -type f \( -name "*.tsx" -o -name "*.jsx" \) \
  ! -path "*/node_modules/*" ! -path "*/.next/*" ! -path "*/dist/*" \
  | sort | head -120

# Project type + deps
cat package.json 2>/dev/null

# File sizes (bundle risk)
find src/ \( -name "*.tsx" -o -name "*.jsx" \) 2>/dev/null \
  | xargs wc -l 2>/dev/null | sort -rn | head -25

# Import graph — who imports what
grep -rn "from '" src/ --include="*.tsx" --include="*.jsx" 2>/dev/null \
  | grep -v node_modules | head -200

# State + context locations
grep -rl "useState\|useReducer\|createContext\|useContext\|zustand\|redux\|jotai" \
  src/ --include="*.tsx" --include="*.ts" 2>/dev/null

# Memoization coverage
grep -rl "React.memo\|useMemo\|useCallback" \
  src/ --include="*.tsx" --include="*.ts" 2>/dev/null

# Lazy / dynamic loading
grep -rl "lazy(\|dynamic(" src/ --include="*.tsx" --include="*.ts" 2>/dev/null

# 'use client' in Next.js
grep -rl "'use client'" src/ --include="*.tsx" 2>/dev/null
```

Read every file found. For each component extract:
- Name, type (page / layout / feature / ui / hook / context / util)
- Local imports → build the dependency edge list
- Props count + complexity
- State declarations (useState, useReducer, external stores)
- useEffect count + dependency arrays
- Memoization present or absent
- Line count (bundle impact proxy)

Build internal map: `ComponentA → [ComponentB, ComponentC, HookX]`

---

## Step 2 — Run All 15 Analysis Checks

### C1 — Dependency Coupling
- Circular imports: A→B→A (crashes or silent bugs)
- Cross-feature coupling: `features/orders` importing directly from `features/users`
- Deep import chains: A→B→C→D→E→F (fragile, hard to refactor)
- God components: single file >300 lines mixing data, UI, and logic
- Barrel file overuse: `index.ts` re-exporting 20+ things (kills tree-shaking)

### C2 — Re-render Risk
- Inline object/array in JSX props: `<C style={{ color: 'red' }}>` — new ref every render
- Inline function props: `<C onClick={() => fn()}>` — new function ref every render
- Context value not memoized: `<Ctx.Provider value={{ a, b }}>` — triggers all consumers
- Missing `React.memo` on expensive children receiving stable props
- Missing `useCallback` on functions passed to memoized children
- Missing `useMemo` on expensive pure computations
- `key={Math.random()}` or `key={index}` on reorderable lists

### C3 — State Architecture
- Prop drilling: same prop passed through 3+ levels without consumption
- Duplicate state: same data in multiple sibling useState calls
- Derived state as useState: `const [full, setFull] = useState(first + last)`
- State that belongs in URL: filters, pagination, active tabs
- Global store used for ephemeral UI state (modal open, tooltip visible)
- Missing loading / error / empty states on async components

### C4 — Bundle Impact
- No route-level code splitting: missing `dynamic()` in Next.js or `React.lazy()`
- Whole-library imports: `import _ from 'lodash'`, `import * as Icons from 'react-icons'`
- Heavy deps in shared components rendered on every page
- `<img>` instead of `next/image`
- Fonts via `<link>` instead of `next/font` (CLS + extra network request)

### C5 — Hook Quality
- Hooks called conditionally or inside loops (Rules of Hooks violation)
- `useEffect` with missing deps (stale closure bug)
- `useEffect` with `[]` that immediately sets state (use useState initializer instead)
- `useEffect` for derived state (use useMemo or inline)
- Custom hooks that mix 3+ unrelated concerns

### C6 — Memory Leaks
- `fetch` in useEffect without AbortController
- `addEventListener` on window/document without cleanup removal
- `setInterval` / `setTimeout` without clearInterval/clearTimeout in cleanup
- WebSocket / SSE not closed on unmount
- Store subscriptions in hooks without unsubscribe

### C7 — Next.js App Router
- `'use client'` on components using no hooks or browser APIs
- useEffect fetch where Server Component fetch would work
- Missing `loading.tsx` / `error.tsx` on route segments
- `useSearchParams()` outside Suspense boundary (hydration crash)
- `searchParams` rendered directly without encoding: `<h1>{params.title}</h1>`
- Server Actions without input validation or auth guard
- Non-serializable props across Server→Client boundary
- PII / secrets passed as props into Client Component tree

### C8 — Performance Anti-patterns
- Heavy synchronous computation in render body
- `useEffect(() => setState(x), [x])` causes double render — derive it instead
- Large lists without virtualization (react-window / tanstack-virtual)
- `router.push` in useEffect for redirects (use server-side `redirect()`)
- Deep conditional rendering chains blocking paint

### C9 — Component Tree Depth
- Max depth > 8 levels
- Components that pass all props through without using any (pass-through wrappers)
- Provider hell: 5+ nested providers at root with no logical grouping

### C10 — Error & Suspense Coverage
- No error boundary around async or heavy components
- `React.lazy()` without wrapping `<Suspense>`
- Unhandled promise rejections in event handlers
- Error states that bubble uncaught to the root

### C11 — Type Safety
- `any` on component props interfaces
- Non-null assertion `!` on API response values
- `useRef(null)` without generic: should be `useRef<HTMLDivElement>(null)`
- Missing return types on custom hooks

### C12 — Accessibility Structure
- `<div onClick>` without role or keyboard handler
- Missing `alt` on images
- Forms without label associations
- Focus not restored after modal / drawer close

### C13 — Dead Code
- Components never imported anywhere in the project
- Props in interface never used in component body
- useEffect with no-op body
- Commented-out JSX blocks in production components

### C14 — Server / Client Boundary
- Server Component → Client Component → Server Component (breaks the boundary)
- `'use client'` at file level when only one of multiple exports needs it
- Sensitive server data passed as props into Client Component subtree

### C15 — Conventions & Naming
- Components not in PascalCase
- Hooks not prefixed with `use`
- Mixed naming schemes in same directory
- Pages not co-located with their feature logic

---

## Step 3 — Build Graph Data

Construct this data mentally to drive the artifact:

```
NODES: id, name, type, filePath, lineCount,
       issues[{ check, severity, summary }],
       severity: critical|high|medium|low|clean,
       rerenderRisk: high|medium|low,
       bundleWeight: heavy|medium|light

EDGES: source → target,
       type: renders|imports|uses-hook|uses-context

CLUSTERS: pages | features | ui | hooks | contexts
```

---

## Step 4 — Outputs

### A — Written Report (in chat)

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🧩 COMPONENT ARCHITECTURE REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📦 Project     : [name]
📁 Components  : [n]  |  Hooks: [n]  |  Contexts: [n]
🌲 Max Depth   : [n] levels
📏 Largest     : [name] ([n] lines)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
HEALTH SCORE  [n] / 100
[██████░░░░░░░░░░░░░░]
🔴 Critical:[n]  🟠 High:[n]  🟡 Medium:[n]  🔵 Low:[n]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FINDINGS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[SEVERITY] [C-ID] — [Title]
📄 [file] (~line N)
🔍 [exact code evidence]
⚠️  [why this causes the specific problem]
✅ [precise fix]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ARCHITECTURE NOTES
[2-4 sentences on coupling, biggest systemic risk, overall quality]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### B — Animated Artifact (MANDATORY)

After the written report, generate a React `.jsx` artifact.
Read `references/artifact-template.md` for the full implementation blueprint.

Embed **actual scanned nodes and edges** from the project.
Follow the visual spec exactly: spring physics, severity colors, flowing edges,
cluster zones, hover tooltips, click-to-highlight, filter panel, health gauge.

---

## Step 5 — After Report

Always end with:

```
──────────────────────────────────────────
Want me to fix any of these?
Tell me which checks (e.g. "fix C2 and C6") and I'll implement the fixes.
──────────────────────────────────────────
```

Fix rules: fix only confirmed items · show inline diff · "fix all" requires list confirmation first

---

## Quick Mode

User pastes 1–5 components → skip bash scan → parse directly → run all 15 checks →
generate artifact with pasted components as nodes → header: `📄 Scanned: pasted code`

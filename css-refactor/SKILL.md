---
name: css-refactor
description: >
  Scan, audit, and refactor CSS across an entire project. Use this skill whenever the user wants to:
  convert CSS ↔ Tailwind, extract reusable utility classes, optimize responsive styles, fix duplicate
  or bloated styles, audit a component or whole codebase for CSS issues, or improve design consistency.
  Trigger on phrases like "refactor CSS", "convert to Tailwind", "CSS to utility classes", "optimize
  responsive", "clean up styles", "audit my components", "too much CSS", "duplicate styles", or any
  request touching styling, theming, or responsive design at scale. Even if the user only mentions one
  file, always offer to scan the full project.
---

# CSS Refactor Skill

Pro-grade CSS auditing and refactoring. Minimal tokens, maximum output quality.

---

## 1. MODE SELECT — ask once, then proceed

On first use, confirm the mode if not obvious from context:

| # | Mode | Trigger phrase examples |
|---|------|------------------------|
| A | **CSS → Tailwind** | "convert to Tailwind", "migrate to utility classes" |
| B | **Tailwind → CSS** | "extract to CSS", "remove Tailwind", "plain CSS" |
| C | **Reusable Classes** | "extract components", "reduce duplication", "DRY" |
| D | **Responsive Audit** | "fix responsive", "mobile issues", "breakpoints" |
| E | **Full Project Audit** | "scan everything", "optimize all styles", "audit project" |

Modes compose — e.g. Mode A + D = convert to Tailwind AND optimize responsive.

---

## 2. SCAN PHASE

Before touching any file, build a mental map:

```
1. List all style files: find . -name "*.css" -o -name "*.scss" -o -name "*.module.css"
2. List all components: find . -name "*.tsx" -o -name "*.jsx" -o -name "*.vue" -o -name "*.html"
3. Detect framework: check package.json for tailwind, styled-components, sass, etc.
4. Detect config: tailwind.config.*, theme tokens, design system files
```

Report findings as a compact table before starting:

```
📦 Project Scan
├── Style files : 12 (.css × 8, .module.css × 4)
├── Components  : 34 (.tsx × 34)
├── Framework   : Next.js + Tailwind v3
├── Issues found: 47 duplicate rules, 12 magic values, 8 breakpoint inconsistencies
└── Mode        : A + D (CSS → Tailwind + Responsive Audit)
```

---

## 3. REFACTOR RULES

### Mode A — CSS → Tailwind

**Strategy:** 1:1 property mapping → semantic grouping → responsive variants

```css
/* BEFORE */
.card {
  display: flex;
  flex-direction: column;
  padding: 16px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  background: #ffffff;
}
@media (min-width: 768px) { .card { flex-direction: row; } }
```

```jsx
// AFTER — semantic Tailwind grouping
<div className="flex flex-col md:flex-row p-4 rounded-lg shadow-md bg-white">
```

**Rules:**
- Group: layout → spacing → visual → interactive → responsive
- Use `@apply` in CSS modules only when className reuse > 3×
- Prefer JIT arbitrary values `[value]` over inline styles
- Flag non-Tailwind values (custom colors, odd spacing) → suggest `extend` config

---

### Mode B — Tailwind → CSS

**Strategy:** Extract class clusters into semantic `.component` names

```jsx
// BEFORE
<button className="px-4 py-2 bg-blue-600 hover:bg-blue-700 text-white font-semibold rounded-md transition-colors disabled:opacity-50 disabled:cursor-not-allowed">
```

```css
/* AFTER — vars + cascade */
.btn-primary {
  padding: 0.5rem 1rem;
  background: var(--color-blue-600);
  color: white;
  font-weight: 600;
  border-radius: 0.375rem;
  transition: background-color 150ms ease;

  &:hover { background: var(--color-blue-700); }
  &:disabled { opacity: 0.5; cursor: not-allowed; }
}
```

**Rules:**
- Output CSS custom properties for all color/spacing tokens
- Use `&` nesting (modern CSS) unless project targets legacy browsers
- Group pseudo-classes inside selector block
- Consolidate modifier variants (hover/focus/active) into one block

---

### Mode C — Reusable Classes / DRY

**Detection patterns (flag these):**
- Same 3+ property block repeated across 2+ selectors → extract
- Magic numbers: `margin: 13px`, `color: #3d4f6a` → replace with token
- Specificity wars: `.parent .child .item` chains → flatten
- Utility bloat: 8+ Tailwind classes on one element → `@apply` component

**Output format:**

```css
/* tokens.css — emit this first */
:root {
  --space-4: 1rem;
  --radius-card: 0.5rem;
  --shadow-card: 0 2px 8px rgb(0 0 0 / 0.1);
  --color-surface: #ffffff;
}

/* components.css */
.card { padding: var(--space-4); border-radius: var(--radius-card); ... }
```

---

### Mode D — Responsive Audit

**Breakpoint consistency check:**

```
Standard scale to enforce (Tailwind defaults — adapt if project has custom):
sm: 640px  |  md: 768px  |  lg: 1024px  |  xl: 1280px  |  2xl: 1536px
```

**Flag these anti-patterns:**
```css
/* ❌ Magic breakpoints */
@media (max-width: 743px) { }   /* non-standard */
@media (min-width: 800px) { }   /* non-standard */

/* ❌ Redundant mobile-first override */
.el { font-size: 16px; }
@media (min-width: 768px) { .el { font-size: 16px; } }  /* same value */

/* ❌ Desktop-first in Tailwind project */
@media (max-width: 768px) { }   /* use sm: prefix instead */
```

**Fix pattern — always mobile-first:**
```css
/* ✅ */
.grid { grid-template-columns: 1fr; }
@media (min-width: 640px) { .grid { grid-template-columns: repeat(2, 1fr); } }
@media (min-width: 1024px) { .grid { grid-template-columns: repeat(3, 1fr); } }
```

---

### Mode E — Full Project Audit

Run all modes. Output a prioritized action plan:

```
🔴 Critical (breaking / major bloat)
  - Button.tsx: 94-class className string → extract to .btn-* components
  - global.css: 340 lines, 60% duplicated from component files

🟡 Moderate (inconsistency / maintainability)  
  - 7 files use px values instead of rem
  - Breakpoints vary: 743px, 768px, 770px → standardize to 768px

🟢 Minor (polish)
  - 12 color literals → suggest CSS custom properties
  - 4 @media blocks can be merged
```

---

## 4. OUTPUT FORMAT

### Single file refactor
Emit the complete refactored file. No partial diffs unless file > 200 lines.

### Multi-file / project
Emit files one at a time with clear headers:
```
=== src/styles/tokens.css ===
[full file content]

=== src/components/Button.tsx ===
[full file content]
```

### Always append a change summary
```
📝 Changes
- Converted 47 CSS rules → Tailwind classes
- Extracted 6 reusable components (@apply)
- Fixed 8 breakpoint inconsistencies → mobile-first
- Replaced 12 magic values → CSS custom properties
- Removed 340 lines of duplicate CSS
```

---

## 5. QUALITY RULES (non-negotiable)

- **No magic numbers** — every value should trace to a token or Tailwind scale
- **Mobile-first always** — `min-width` queries, never `max-width` in new code
- **Semantic class names** — `.card`, `.btn-primary`, not `.div-blue-box`
- **Preserve all behavior** — never remove styles that affect layout/interaction without flagging
- **Flag breaking changes** — if a refactor changes visual output, say so explicitly

---

## 6. REFERENCE FILES

Load when needed:
- `references/tailwind-map.md` — full CSS property → Tailwind class mapping
- `references/responsive-patterns.md` — common responsive patterns + fixes

---

## 7. QUICK MODE (token-efficient for small tasks)

If user pastes a single component/snippet (< 50 lines), skip the scan phase.
Go directly: detect current style → apply mode → output refactored code + change summary.

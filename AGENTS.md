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
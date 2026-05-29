---
name: frontend-security-scanner
description: >
  Scan frontend projects for security vulnerabilities and generate a detailed report directly
  in chat. Use this skill whenever the user wants to audit, check, scan, or review their
  frontend code for security issues — even if they don't say "security scan". Triggers include:
  "check my code for security issues", "scan for XSS", "any exposed keys?", "security audit",
  "is my code safe?", "check for vulnerabilities", "review my project security", "find security
  bugs", "check env vars", "is dangerouslySetInnerHTML safe here", "review my auth code",
  "check my API calls", "scan my components", or any mention of security, vulnerabilities,
  exploits, or unsafe patterns in frontend code. Always output the report in chat — never
  create files unless the user explicitly asks. Never auto-fix issues — always ask first.
---

# Frontend Security Scanner

Scan frontend codebases for security vulnerabilities. Output a structured report **in chat only**.

## Core Rules

1. **REPORT ONLY** — never create files, never auto-fix, never modify code
2. **ASK BEFORE FIXING** — after the report, ask: *"Want me to fix any of these?"*
3. **If user says fix** — implement only the specific issues they confirm, one at a time
4. **Scan scope** — whole project by default; single file/component if user pastes code

---

## Scan Workflow

### Step 1 — Scope Check

If the user pastes code → scan it directly, skip to Step 3.

If the user says "scan my project" → run these bash commands first:

```bash
# Map the project
find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" \) \
  ! -path "*/node_modules/*" ! -path "*/.next/*" ! -path "*/dist/*" ! -path "*/build/*" \
  | head -80

# Check for .env files
find . -name ".env*" ! -path "*/node_modules/*" | head -20

# Check package.json for dep versions
cat package.json 2>/dev/null | grep -E '"(next|react|axios|jwt|crypto|helmet|sanitize)"'
```

### Step 2 — Read Files

Read all source files found. Prioritize:
- `src/` components, pages, hooks, utils
- API client files (`lib/api/`, `services/`)
- Auth-related files
- Any file with `env`, `key`, `token`, `secret`, `auth` in the name
- Config files (`next.config.*`, `vite.config.*`)

### Step 3 — Run All 12 Security Checks

Go through every check below against the code. Track every finding with: file path, line reference, severity, and evidence snippet.

---

## Security Checks

### C1 — XSS (Cross-Site Scripting)

**What to find:**
- `dangerouslySetInnerHTML={{ __html: userInput }}` — direct injection
- `dangerouslySetInnerHTML` with any variable that originates from API, URL, or user input
- `innerHTML =` or `outerHTML =` in vanilla JS / useEffect
- `document.write()`
- Template literals injected into DOM: `` el.innerHTML = `...${variable}...` ``
- `eval()`, `new Function()`, `setTimeout(string)`, `setInterval(string)`
- Unescaped output in server-rendered HTML (Next.js `getServerSideProps` → JSX)

**Safe patterns (don't flag):**
- `dangerouslySetInnerHTML` with a static string literal
- Content run through DOMPurify, sanitize-html, or xss library before use

---

### C2 — Exposed Secrets & API Keys

**What to find:**
- Hardcoded strings matching key patterns:
  - `sk-[a-zA-Z0-9]{20,}` (OpenAI)
  - `AIza[0-9A-Za-z\-_]{35}` (Google)
  - `AKIA[0-9A-Z]{16}` (AWS)
  - `ghp_[a-zA-Z0-9]{36}` (GitHub PAT)
  - `xoxb-|xoxp-` (Slack)
  - Any variable named `SECRET`, `PRIVATE_KEY`, `API_KEY`, `PASSWORD`, `TOKEN` assigned a non-env string
- Secrets in `next.config.*`, `vite.config.*`, committed `.env` files
- `NEXT_PUBLIC_` or `VITE_` prefixed vars holding secrets (client-exposed by design — flag if value looks sensitive)
- Auth tokens stored in `localStorage` or `sessionStorage`

---

### C3 — Unsafe Rendering & Injection

**What to find:**
- React `ref.current.innerHTML = value` where value is dynamic
- Server Actions / API routes returning unsanitized user input directly into rendered HTML
- SVG injection: user-controlled SVG rendered without sanitization
- CSS injection: user-controlled style values (`style={{ color: userInput }}`)
- URL injection: `<a href={userInput}>` or `<iframe src={userInput}>` without validation — allows `javascript:` URIs
- `<script src={dynamicUrl}>` or dynamic script tag creation

---

### C4 — LocalStorage / SessionStorage Misuse

**What to find:**
- Auth tokens, JWTs, session IDs stored in localStorage (XSS-accessible)
- Sensitive PII stored in localStorage (credit cards, SSN, passwords)
- Data stored in localStorage without any expiry logic
- `JSON.parse(localStorage.getItem(...))` without try/catch — can throw + crash app
- Storing large objects that could cause storage quota issues

**Preferred patterns:**
- Auth tokens → `httpOnly` cookies (not accessible to JS)
- Non-sensitive state → localStorage is fine (UI preferences, theme, etc.)

---

### C5 — Insecure API Calls

**What to find:**
- `http://` URLs hardcoded (non-HTTPS) in production API endpoints
- Missing auth headers on calls to authenticated endpoints
- API keys in query string params: `fetch('/api?key=abc123')` — logged by servers
- Axios/fetch with `withCredentials: false` on cross-origin calls that need cookies
- No timeout on fetch/axios calls — DoS via hanging requests
- Error responses forwarded directly to UI: `setError(err.response.data)` — can leak stack traces
- CORS set to `*` in API route handlers (Next.js API routes / Express)

---

### C6 — Authentication & Authorization

**What to find:**
- Client-side only auth checks (`if (user.role === 'admin') showAdminPanel`) without server-side guard
- JWT decoded and trusted on the client without server verification
- `localStorage.getItem('isLoggedIn') === 'true'` used as auth guard
- Missing auth check on Next.js API routes or Server Actions
- Passwords stored or logged anywhere on the frontend
- Weak redirect after login: `router.push(searchParams.get('redirect'))` without validating the redirect URL (open redirect)
- Session tokens not cleared on logout

---

### C7 — Dependency Vulnerabilities

**What to find:**
- Known vulnerable package versions (check against common CVEs):
  - `axios < 1.6.0` — SSRF vulnerability
  - `next < 14.1.1` — path traversal (CVE-2024-34351)
  - `next < 13.5.7` — open redirect
  - `lodash < 4.17.21` — prototype pollution
  - `minimist < 1.2.6` — prototype pollution
  - `node-fetch < 2.6.7` — SSRF
  - `jsonwebtoken < 9.0.0` — algorithm confusion
- `npm audit` style check: flag if `package.json` has dependencies without lock file

---

### C8 — Environment Variable Leakage

**What to find:**
- `NEXT_PUBLIC_` vars containing secrets (secret keys, DB passwords, internal service URLs)
- `VITE_` vars containing secrets
- `console.log(process.env)` or `console.log(import.meta.env)` — dumps all env vars
- Env vars logged in error handlers
- API secret keys passed to client components in Next.js App Router (prop drilling from Server Component to Client Component)
- `.env.local`, `.env.production` files not in `.gitignore`

---

### C9 — Content Security Policy (CSP)

**What to find:**
- No CSP headers configured (check `next.config.*` headers, `_document.tsx`, or middleware)
- CSP with `unsafe-inline` scripts — defeats XSS protection
- CSP with `unsafe-eval` — allows eval-based attacks
- CSP with `*` wildcard for `script-src` or `connect-src`
- Missing security headers: `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`

**Check locations:** `next.config.*` headers array, `middleware.ts`, `_document.tsx` meta tags.

---

### C10 — Prototype Pollution & Object Injection

**What to find:**
- `Object.assign({}, userInput)` without sanitization
- Spread operator on unchecked API data: `{ ...req.body }` into sensitive objects
- Dynamic key assignment: `obj[userInput] = value` where userInput could be `__proto__`
- `JSON.parse` on untrusted input used directly in object merge operations
- `lodash.merge`, `_.set` with user-controlled paths

---

### C11 — Third-Party Script & Supply Chain Risk

**What to find:**
- External scripts loaded without `integrity` (SRI hash): `<Script src="https://cdn.example.com/lib.js">`
- CDN scripts with version ranges instead of pinned versions: `@latest`, `@3.x`
- `dangerouslySetInnerHTML` used to inject third-party ad/analytics HTML
- Dynamic `import()` of URLs: `import(userControlledUrl)`
- postMessage handlers without `event.origin` validation

---

### C12 — React / Framework-Specific Issues

**What to find:**
- `useEffect` fetching with user-controlled URL without validation
- `key={index}` on lists where items can be reordered — can cause state leakage between items (not security-critical but flag as low)
- Uncontrolled components with `defaultValue` set from URL params
- Next.js `cookies()` / `headers()` called in Client Components (throws + leaks server context)
- Server Actions without input validation or CSRF protection
- `revalidatePath('/')` called without auth check in Server Actions — cache poisoning
- Directly rendering `searchParams` values without encoding: `<h1>{searchParams.title}</h1>`

---

## Report Format

Output this exact structure in chat. Omit sections with zero findings.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔐 FRONTEND SECURITY SCAN REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📁 Scanned: [file count] files  |  [date]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SUMMARY
───────
🔴 Critical : [n]
🟠 High     : [n]
🟡 Medium   : [n]
🔵 Low      : [n]
⚪ Info      : [n]
───────────────
Total        : [n]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FINDINGS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[SEVERITY] [CHECK-ID] — [Short Title]
📄 File: src/components/Example.tsx (line ~42)
🔍 Found: `dangerouslySetInnerHTML={{ __html: comment }}`
⚠️  Risk: comment comes from user input via API — direct XSS vector
✅ Fix:  Sanitize with DOMPurify before rendering, or use a text node

— (repeat for each finding) —

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PASSED CHECKS ✅
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ C2  No hardcoded secrets found
✅ C5  All API calls use HTTPS
... (list checks with zero findings)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Severity Levels

| Level | When to use |
|---|---|
| 🔴 Critical | Direct exploit possible: RCE, auth bypass, hardcoded prod secret, SQLi |
| 🟠 High | XSS, JWT misuse, open redirect, sensitive data exposure |
| 🟡 Medium | Missing CSP, localStorage auth tokens, missing input validation |
| 🔵 Low | Missing SRI, no timeout on fetch, index keys on lists |
| ⚪ Info | Best practice deviation, minor code smell with security implication |

---

## After the Report

Always end with exactly this prompt (no exceptions):

```
───────────────────────────────────────
Want me to fix any of these issues?
Tell me which ones (e.g. "fix C1 and C6") and I'll implement the fixes.
───────────────────────────────────────
```

If user says fix → implement only what they asked for, show the diff/changed code inline. Do not touch other files.

If user says "fix all" → confirm the list first: *"I'll fix: [C1, C3, C6]. Proceed?"*

---

## Quick Scan Mode

If user pastes a single file or snippet (no project context):
- Skip Step 1 and 2
- Run all 12 checks on the pasted code
- Output the same report format but with `📄 Scanned: pasted code` in the header
- Takes priority over project scan if code is in the message

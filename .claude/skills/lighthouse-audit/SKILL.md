---
name: lighthouse-audit
description: >
  Run professional Lighthouse performance, SEO, accessibility, and best-practices audits on any web project.
  Use this skill whenever the user mentions "Lighthouse", "SEO audit", "performance report", "page speed",
  "web vitals", "accessibility audit", "site audit", or wants to analyze, score, or improve a web project's
  frontend quality. Triggers on: "run Lighthouse", "audit my site", "check SEO", "performance audit",
  "generate report", "improve page speed", "check Core Web Vitals", "what should I fix for SEO",
  "accessibility issues", or any request to analyze a web project's performance/quality.
  Even if the user just says "check my project" or "what needs fixing" in a web project context — use this skill.
  All work is done in temp files only; nothing is permanently written to the project.
---

# Lighthouse Audit Skill

You are a **senior frontend performance engineer**. You produce professional, data-driven Lighthouse audit
reports with prioritized, developer-ready recommendations. You work only in **temp files** — never write
anything permanent to the user's project. The final report is delivered in chat only.

---

## STEP 1 — Mode Selection (always first)

Use `ask_user_input_v0`:

```
"Which pages would you like to audit?"
Options:
  1. Entire Project — All Pages
  2. Select Specific Pages
```

---

## STEP 2 — Page Discovery

### A. Check for CHANGELOG.md first (token-efficient)

```bash
cat CHANGELOG.md 2>/dev/null | head -200
```

Extract route/page references from it — look for "Added /page", "New route", "Feature: /section" patterns.

### B. If no CHANGELOG or routes not found, scan the project efficiently

Run these — never `cat` whole source files:

```bash
# Detect framework
ls package.json next.config.* vite.config.* angular.json nuxt.config.* 2>/dev/null

# Next.js App Router
find ./app -name "page.tsx" -o -name "page.jsx" -o -name "page.js" 2>/dev/null \
  | sed 's|./app||;s|/page\.[jt]sx\?||' | sort

# Next.js Pages Router
find ./pages -name "*.tsx" -o -name "*.jsx" -o -name "*.js" 2>/dev/null \
  | grep -v "_app\|_document\|api/" | sed 's|./pages||;s|\.[jt]sx\?||' | sort

# React Router / Vue Router
grep -r "path:" src/ --include="*.ts" --include="*.js" -h 2>/dev/null \
  | grep -oE "'[/][^']+'" | tr -d "'" | sort -u

# Sitemap
grep -oE "<loc>[^<]+</loc>" public/sitemap.xml 2>/dev/null | sed 's|<loc>||;s|</loc>||'

# Static HTML
find . -maxdepth 3 -name "*.html" ! -path "*/node_modules/*" ! -path "*/.next/*" 2>/dev/null
```

---

## STEP 3 — Page Selection (Mode 2 only)

After discovery, present found pages with `ask_user_input_v0` as `multi_select`.

---

## STEP 4 — Run Lighthouse

### Pre-flight (write to /tmp — never to project)

```bash
# Check tools
which lighthouse 2>/dev/null || npm install -g lighthouse --silent
which chromium-browser google-chrome chromium 2>/dev/null | head -1

# Start local server if project not running
# Check package.json for start script
grep -E '"(dev|start|serve|preview)"' package.json 2>/dev/null
```

### Run audit — capture output directly to variable, no files created

```bash
lighthouse "<URL>" \
  --output=json \
  --output-path=stdout \
  --chrome-flags="--headless --no-sandbox --disable-gpu --disable-dev-shm-usage" \
  --only-categories=performance,accessibility,best-practices,seo \
  --quiet \
  --no-update-notifier
```

- Pipe stdout directly into the node parse command (see STEP 5)
- For **local projects**: start server in background (`npm run dev &`), wait for port, then use `http://localhost:<port>`
- For **multiple pages**: run once per page, capture each stdout separately
- **No files or folders are created anywhere**

---

## STEP 5 — Parse Results (pipe from stdout, no files)

```bash
lighthouse "<URL>" --output=json --output-path=stdout \
  --chrome-flags="--headless --no-sandbox --disable-gpu" \
  --only-categories=performance,accessibility,best-practices,seo \
  --quiet --no-update-notifier | node -e "
const chunks = [];
process.stdin.on('data', d => chunks.push(d));
process.stdin.on('end', () => {
  const r = JSON.parse(Buffer.concat(chunks));
  const c = r.categories, a = r.audits;
  console.log('SCORES:', JSON.stringify({
    perf: Math.round(c.performance.score*100),
    seo:  Math.round(c.seo.score*100),
    a11y: Math.round(c.accessibility.score*100),
    bp:   Math.round(c['best-practices'].score*100)
  }));
  console.log('CWV:', JSON.stringify({
    lcp:  a['largest-contentful-paint']?.displayValue,
    cls:  a['cumulative-layout-shift']?.displayValue,
    tbt:  a['total-blocking-time']?.displayValue,
    fcp:  a['first-contentful-paint']?.displayValue,
    ttfb: a['server-response-time']?.displayValue,
  }));
  Object.entries(a).forEach(([id,v]) => {
    if (v.score !== null && v.score < 1 && v.scoreDisplayMode !== 'informative')
      console.log('FAIL:', id, v.score?.toFixed(2), v.displayValue||'', v.title);
  });
});
"
```

**Tier each failing audit:**

- 🔴 **Critical** → score < 0.50 OR Core Web Vital OR missing meta/title/canonical
- 🟡 **Important** → score 0.50–0.89
- 🟢 **Passed** → score 1.0

---

## STEP 6 — Build & Output the Report (in chat + download files)

Construct the report in your response using this exact structure.
After printing it in chat, you will also generate downloadable files (see STEP 7).

---

````
# 🔍 Lighthouse Audit Report
**Project:** <name> | **Date:** <today> | **URL:** <url>

---

## 📊 Score Dashboard

| Category       | Score | Status |
|----------------|-------|--------|
| Performance    |  XX   |  🔴/🟡/🟢 |
| SEO            |  XX   |  🔴/🟡/🟢 |
| Accessibility  |  XX   |  🔴/🟡/🟢 |
| Best Practices |  XX   |  🔴/🟡/🟢 |

Score bands: 🟢 90–100 (Good)  🟡 50–89 (Needs Work)  🔴 0–49 (Poor)

---

## ⚡ Core Web Vitals

| Metric | Value | Target  | Status |
|--------|-------|---------|--------|
| LCP    |       | < 2.5s  | 🔴/🟢  |
| CLS    |       | < 0.1   | 🔴/🟢  |
| TBT    |       | < 200ms | 🔴/🟢  |
| FCP    |       | < 1.8s  | 🔴/🟢  |
| TTFB   |       | < 800ms | 🔴/🟢  |

---

## 🚨 Critical Issues  [N found — fix these first]

### 1. <Audit Title>
- **Score:** XX/100
- **Current:** <displayValue>
- **Why it matters:** <one sentence from audit description>
- **Top offenders:** <list files/elements if available in audit items>
- **Fix:**
  ```<lang>
  // concrete before/after code example
````

[repeat for each critical]

---

## ⚠️ Important Issues [N found]

### 1. <Audit Title>

- **Score:** XX/100
- **Current:** <displayValue>
- **Fix:** <one-liner or short code snippet>

[repeat for each important]

---

## ✅ Passing Audits

<comma-separated list of passed audit titles>

---

## 🛠️ Prioritized Action Plan

### ⚡ Quick Wins (< 1 hour each)

1. ...

### 🔧 Medium Effort (half-day)

1. ...

### 🏗️ Long-term (planned sprint)

1. ...

---

> 💡 **Report complete.** To apply fixes say: **"apply the critical fixes"** or **"fix [specific issue name]"**

````

---

## Code fix patterns (use inline when writing the report)

Reference these patterns when writing fix examples in the report. Do NOT cite this section — just use the patterns.

**render-blocking-resources**
```html
<link rel="preload" as="style" href="/styles.css" onload="this.onload=null;this.rel='stylesheet'">
<script src="/app.js" defer></script>
````

**uses-webp-images / uses-optimized-images**

```html
<picture>
  <source srcset="/img/hero.avif" type="image/avif" />
  <source srcset="/img/hero.webp" type="image/webp" />
  <img src="/img/hero.jpg" alt="..." width="1200" height="600" loading="lazy" />
</picture>
```

**offscreen-images** → `<img loading="lazy">`

**unused-javascript** → dynamic imports: `const C = dynamic(() => import('./C'))`

**unused-css-rules** → enable Tailwind purge in `content: ['./src/**/*.{js,jsx,ts,tsx}']`

**uses-rel-preconnect** → `<link rel="preconnect" href="https://fonts.googleapis.com">`

**font-display** → `font-display: swap;` in every `@font-face`

**uses-text-compression** → `app.use(require('compression')())` or nginx `gzip on;`

**meta-description**

```html
<meta name="description" content="150–160 char unique description per page" />
<!-- Next.js App Router: export const metadata = { description: '...' } -->
```

**document-title** → `<title>Page | Brand</title>`

**canonical** → `<link rel="canonical" href="https://example.com/page">`

**structured-data**

```html
<script type="application/ld+json">
  { "@context": "https://schema.org", "@type": "WebPage", "name": "...", "url": "..." }
</script>
```

**image-alt** → `<img alt="descriptive text">` or `alt=""` for decorative

**color-contrast** → minimum 4.5:1 for body text, 3:1 for large text

**button-name** → `<button aria-label="Close">✕</button>`

**efficient-animated-content** → replace GIF with `<video autoplay loop muted playsinline>`

**no-document-write** → replace with `document.createElement` + `appendChild`

**server-response-time** → add CDN, enable SSG/ISR, or add HTTP caching headers

---

## STEP 7 — Inline Download Buttons (show_widget, zero filesystem writes)

After printing the report in chat, call the `visualize:show_widget` tool to render two download buttons.
Build the full MD and HTML content as JS template literal strings inside the widget script.
Use `URL.createObjectURL(new Blob([...]))` — no files, no folders, no present_files anywhere.

### Exact widget pattern to use:

```html
<h2 class="sr-only">Lighthouse report downloads</h2>
<div style="padding:1rem 0;display:flex;gap:12px;flex-wrap:wrap" id="lh-btns"></div>
<script>
  const PROJECT = '<project-name>';
  const DATE = '<YYYY-MM-DD>';

  const md = `<FULL MARKDOWN REPORT STRING — same content shown in chat, escaped for JS template literal>`;

  const html = `<!DOCTYPE html><html lang="en"><head><meta charset="UTF-8"><title>Lighthouse Report — ${PROJECT}</title><style>body{font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',sans-serif;background:#0f1117;color:#e2e8f0;padding:2rem;line-height:1.6}h1{font-size:1.6rem;font-weight:500;margin-bottom:.25rem}h2{font-size:1.1rem;font-weight:500;margin:2rem 0 .75rem;padding-bottom:.3rem;border-bottom:1px solid #2d3148}.meta{color:#718096;font-size:.85rem;margin-bottom:2rem}.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(140px,1fr));gap:1rem;margin-bottom:2rem}.card{background:#1a1d27;border:1px solid #2d3148;border-radius:12px;padding:1.25rem;text-align:center}.label{font-size:.7rem;color:#718096;text-transform:uppercase;letter-spacing:.05em;margin-bottom:.5rem}.score{font-size:2.2rem;font-weight:500}.red{color:#e53e3e}.yellow{color:#d69e2e}.green{color:#38a169}table{width:100%;border-collapse:collapse;margin-bottom:1.5rem;font-size:.875rem}th{background:#1a1d27;padding:.5rem .75rem;text-align:left;font-size:.7rem;text-transform:uppercase;color:#718096;border:1px solid #2d3148}td{padding:.5rem .75rem;border:1px solid #2d3148}.issue{background:#1a1d27;border:1px solid #2d3148;border-radius:8px;padding:1rem;margin-bottom:.75rem}.critical{border-left:3px solid #e53e3e}.important{border-left:3px solid #d69e2e}pre{background:#0d0f18;border:1px solid #2d3148;border-radius:6px;padding:.75rem;overflow-x:auto;font-size:.8rem;margin:.5rem 0 0}code{font-family:monospace}.footer{margin-top:3rem;padding-top:1rem;border-top:1px solid #2d3148;color:#718096;font-size:.75rem;text-align:center}</style></head><body>
<h1>Lighthouse Audit Report</h1>
<p class="meta"><strong>Project:</strong> ${PROJECT} &nbsp;|&nbsp; <strong>Date:</strong> ${DATE}</p>
<INSERT FULL HTML BODY CONTENT HERE — score cards, CWV table, issues, action plan — built from audit data>
<div class="footer">Generated by Claude Lighthouse Audit Skill &middot; ${DATE}</div>
</body></html>`;

  function makeBtn(label, content, filename, type) {
    const url = URL.createObjectURL(new Blob([content], { type }));
    const a = document.createElement('a');
    a.href = url;
    a.download = filename;
    a.textContent = label;
    a.style.cssText =
      'display:inline-flex;align-items:center;gap:6px;padding:10px 18px;background:var(--color-background-secondary);border:0.5px solid var(--color-border-secondary);border-radius:var(--border-radius-md);color:var(--color-text-primary);text-decoration:none;font-size:14px;font-weight:500;cursor:pointer';
    a.onmouseenter = () => (a.style.background = 'var(--color-background-tertiary)');
    a.onmouseleave = () => (a.style.background = 'var(--color-background-secondary)');
    return a;
  }

  const wrap = document.getElementById('lh-btns');
  wrap.appendChild(
    makeBtn('⬇ Download .md', md, `lighthouse-${PROJECT}-${DATE}.md`, 'text/markdown')
  );
  wrap.appendChild(
    makeBtn('⬇ Download .html', html, `lighthouse-${PROJECT}-${DATE}.html`, 'text/html')
  );
</script>
```

**Rules — strictly follow:**

- NEVER write to `/tmp`, `/mnt/user-data/outputs/`, or any path in the project
- NEVER call `present_files`
- NEVER create any folder or file on the filesystem
- All content lives in JS strings inside the widget script only
- Blob URLs are created in the browser — clicking downloads straight to user's machine
- Escape backticks and `${` inside template literals with `\`` and `\${`

---

## STEP 8 — STOP after report + download buttons

End with:

> 💡 **Report complete.** Use the download buttons above to save your report.
> To apply fixes say: **"apply the critical fixes"** or **"fix [specific issue name]"**

---

## STEP 9 — Apply Fixes (only when user explicitly asks)

When the user asks to fix something:

1. **Confirm** which fix(es) to apply, quoting the audit title
2. **Show a diff** of every change before writing:

   ```
   // BEFORE
   <old code>

   // AFTER
   <new code>
   ```

3. **Ask for confirmation** before writing to any project file
4. **Write directly** to the project file (not /tmp)
5. **Re-run Lighthouse** on the affected page(s) using /tmp output
6. **Show before → after score delta** in chat

---

## Quality Rules

- **Never guess scores** — always parse from actual Lighthouse stdout JSON
- **Never fabricate audit items** — only report what Lighthouse flagged
- **Always show code** — every recommendation includes a concrete before/after example
- **Be specific** — "reduce hero.jpg from 2.3 MB to < 200 KB using WebP" not "optimize images"
- **Rank by ROI** — highest-impact items always listed first
- **Token efficiency** — use `grep`/`find` for discovery, pipe Lighthouse to node inline, never `cat` large files
- **Zero filesystem writes** — no folders, no temp files, no project files touched until user explicitly requests fixes
- **Downloads via blob** — MD and HTML delivered as in-chat blob download buttons only

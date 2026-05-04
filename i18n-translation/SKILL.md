---
name: i18n-translation
description: >
  Simple i18n translation setup for React/Next.js using i18next. Language switching via
  localStorage — NO URL path changes, no /en/ /hi/ routing, no middleware. Generates
  src/i18n/ module (config, types, utils) + locale JSON files + a drop-in LocaleSwitcher
  component. NEVER modifies existing files. Use whenever user mentions i18n, translation,
  language switching, locale JSON, adding a new language, translate my app, missing keys,
  multi-language support, or any .json locale files.
---

# i18n Translation Skill

Simple language switching — **no URL changes, no routing, no middleware.**
Switch language → UI updates in place → choice saved to localStorage.

---

## Decision Gate — Ask First (two options only)

> **Which situation?**
>
> 1. **Set up** — i18n not installed yet, generate everything
> 2. **Already set up** — i18n configured, sync/fill translations now

Then follow the matching workflow below.

---

## ⚠️ STRICT RULE — ALL WORKFLOWS

```
NEVER modify, overwrite, or delete any existing file.
ONLY create brand-new files.
For files the user must edit themselves, print a "Manual Steps" block.
```

---

## Workflow 1 — Set Up

### Step 1 — Detect stack

```bash
cat package.json | grep -E '"next"|"vite"|"react"'
```

### Step 2 — Ask user

- Which languages? (always include `en`)
- Which namespaces? (default: `common`, `auth`, `dashboard`)

### Step 3 — Install

```bash
npm install i18next react-i18next
```

No HTTP backend needed — JSON files are imported directly (Vite/webpack bundled).

### Step 4 — Create these files (check existence first)

```bash
ls src/i18n/ 2>/dev/null || echo "safe"
```

**Files to create:**

```
src/i18n/
├── config.ts          ← i18next init + language list
├── index.ts           ← re-exports
├── types.ts           ← typed key paths
├── utils.ts           ← formatDate, formatNumber, changeLanguage
└── locales/
    ├── en/
    │   ├── common.json
    │   ├── auth.json
    │   └── dashboard.json
    ├── hi/
    │   └── ...same files...
    └── gu/
        └── ...same files...

src/components/
└── LocaleSwitcher.tsx  ← drop-in switcher, no routing
```

Read `references/templates.md` for exact file contents.
Read `references/locale-json.md` for JSON examples (en/hi/gu).

### Step 5 — Print Manual Steps block

```
✅ Created: src/i18n/config.ts
✅ Created: src/i18n/index.ts
✅ Created: src/i18n/types.ts
✅ Created: src/i18n/utils.ts
✅ Created: src/i18n/locales/en/common.json  (+ auth, dashboard)
✅ Created: src/i18n/locales/hi/common.json  (+ auth, dashboard)
✅ Created: src/components/LocaleSwitcher.tsx

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 MANUAL STEPS — add to your existing files:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

── src/main.tsx  OR  src/index.tsx ──
  import './i18n';   ← add at top before ReactDOM

── (optional) src/App.tsx ───────────
  import { Suspense } from 'react';
  // wrap root in: <Suspense fallback="…"><App /></Suspense>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Workflow 2 — Already Set Up

### Step 1 — Find locale folder (token-efficient, stop at first hit)

```bash
# 1. Check CHANGELOG first — may tell you exactly which files changed
head -60 CHANGELOG.md 2>/dev/null

# 2. Check git log — cheapest way to find recently changed en/ files
git log --oneline --since="30 days ago" -- "**/locales/en/**" "**/en/**" 2>/dev/null | head -15

# 3. Only if above gave nothing — find locale folder directly (never scan full project)
find . -type d -name "en" \
  -not -path "*/node_modules/*" \
  -not -path "*/.git/*" 2>/dev/null | head -5
```

Use whichever path is found. Common locations: `src/i18n/locales/`, `public/locales/`, `src/locales/`, `messages/`.

### Step 2 — List all locale folders and en/ files

```bash
# List sibling lang folders (everything next to en/)
ls [LOCALE_BASE_DIR]/

# List all JSON files in en/
ls [LOCALE_BASE_DIR]/en/
```

### Step 3 — Deep diff: compare every lang folder against en/

Run once to get a full picture of ALL missing keys across ALL languages and ALL files:

```bash
node -e "
const fs=require('fs'),path=require('path');
const base='[LOCALE_BASE_DIR]';
const enDir=path.join(base,'en');
const langs=fs.readdirSync(base).filter(l=>l!=='en'&&fs.statSync(path.join(base,l)).isDirectory());
const flat=(o,p='')=>Object.keys(o).flatMap(k=>typeof o[k]==='object'&&o[k]!==null?flat(o[k],p+k+'.'):p+k);
const getVal=(o,k)=>k.split('.').reduce((c,p)=>c?.[p],o);
for(const lang of langs){
  for(const f of fs.readdirSync(enDir).filter(f=>f.endsWith('.json'))){
    const en=JSON.parse(fs.readFileSync(path.join(enDir,f),'utf8'));
    let loc={};try{loc=JSON.parse(fs.readFileSync(path.join(base,lang,f),'utf8'));}catch{}
    const enKeys=flat(en);
    const locKeys=new Set(flat(loc));
    const missingKeys=enKeys.filter(k=>!locKeys.has(k));
    const diffValues=enKeys.filter(k=>locKeys.has(k)&&getVal(en,k)===getVal(loc,k)&&typeof getVal(en,k)==='string');
    if(missingKeys.length) console.log('['+lang+']['+f+'] MISSING('+missingKeys.length+'):', missingKeys.join(', '));
    if(diffValues.length) console.log('['+lang+']['+f+'] UNTRANSLATED('+diffValues.length+'):', diffValues.slice(0,5).join(', '));
  }
}
"
```

This reports:

- **MISSING** — key exists in `en/` but not in the lang file at all
- **UNTRANSLATED** — key exists but value is identical to English (was never translated)

### Step 4 — Translate and fill gaps

For each lang + file that has missing or untranslated keys:

- Read the full `en/[file].json`
- Read the existing `[lang]/[file].json` (or start with `{}` if file missing entirely)
- Translate only the missing/untranslated keys into the target language
- Preserve all existing correct translations — never re-translate them
- Preserve all `{{interpolation}}` vars, key names, nesting structure
- Deep-merge new translations into existing file content
- Write the merged result back to `[lang]/[file].json`

### Step 5 — Report what was done

```
✅ hi/common.json   — added 3 missing keys, translated 2 untranslated
✅ gu/auth.json     — created (was missing entirely)
✅ hi/dashboard.json — no changes needed
```

---

## Pluralization

| Language           | Forms                                              |
| ------------------ | -------------------------------------------------- |
| en, hi, gu, de, es | `_one`, `_other`                                   |
| ru, uk             | `_one`, `_few`, `_many`, `_other`                  |
| ar                 | `_zero`, `_one`, `_two`, `_few`, `_many`, `_other` |
| zh, ja, ko         | `_other` only                                      |

i18next format: `"items_one": "{{count}} item"`, `"items_other": "{{count}} items"`

---

## References — load only when needed

- `references/templates.md` — config.ts, index.ts, types.ts, utils.ts file contents
- `references/locale-json.md` — en/hi/gu JSON examples for all 3 namespaces
- `references/locale-switcher.md` — LocaleSwitcher component (no routing)

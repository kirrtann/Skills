---
name: audit-a11y
description: Run a full accessibility audit on specified components or pages. Checks WCAG 2.1 AA compliance, keyboard navigation, ARIA roles, color contrast, and translation coverage.
---

Run an accessibility and i18n audit for: $ARGUMENTS

@i18n-a11y Audit the specified files (or all components in src/components/ if none given) for:

1. **WCAG 2.1 AA violations** — cite each by criterion number
2. **Keyboard navigation** — all interactive elements reachable and operable
3. **ARIA usage** — correct roles, labels, live regions
4. **Focus management** — visible indicators, logical order, modal traps
5. **Images** — descriptive alt text or empty alt for decorative
6. **Forms** — labels associated, errors linked via aria-describedby
7. **i18n coverage** — any hardcoded strings that should be in translation files

For each issue produce:
- File + line number
- WCAG criterion
- Before (broken) code
- After (fixed) code

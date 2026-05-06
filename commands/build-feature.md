---
name: build-feature
description: Full feature build pipeline. Runs admin decomposition → ui-architect design → react-dev implementation → parallel testing/perf/a11y/code-quality review. Use for any new feature from scratch.
---

Run the full multi-agent feature pipeline for: $ARGUMENTS

## Step 1 — Admin decomposition
@admin Break down this feature request and produce a structured task plan with agent assignments and acceptance criteria:

"$ARGUMENTS"

## Step 2 — UI Architecture  
@ui-architect Using the task plan from Step 1, produce the component tree and full component specs.

## Step 3 — Implementation
@react-dev Using the component specs from Step 2, implement all components, hooks, and store slices. Follow all conventions in CLAUDE.md.

## Step 4 — Quality gates (run in parallel)
@testing Write Playwright E2E tests and Vitest unit tests for the implemented feature.
@i18n-a11y Audit the implemented components for WCAG 2.1 AA violations and add next-intl translations.
@code-quality Review all new files against project standards and produce a scored review report.

## Step 5 — Performance check
@perf Analyze the new feature for Core Web Vitals impact and suggest any needed optimizations.

## Step 6 — Admin final review
@admin Review all outputs from Steps 3–5. Approve or return specific tasks for revision.

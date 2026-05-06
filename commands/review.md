---
name: review
description: Code quality review of specified files or the current git diff. Runs code-quality agent and optionally i18n-a11y and perf agents.
---

Review the following for code quality issues: $ARGUMENTS

If no arguments given, review all files in the current git diff (`git diff --name-only HEAD`).

@code-quality Review each file and produce a scored report (Correctness / Readability / Type safety / Standards / Testability). Flag all issues with file + line number. Mark each file as APPROVED or NEEDS_REVISION.

After code-quality completes:
@i18n-a11y Check the same files for any accessibility violations or missing translation keys.

Produce a combined review summary at the end:
- Files approved: [list]
- Files needing revision: [list with specific issues]
- Action items for @react-dev: [prioritized list]

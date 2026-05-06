---
name: agent-status
description: List all available agents, their roles, and how to invoke them. Shows the full team roster with @-mention syntax and slash commands.
---

Show the full multi-agent team status for this project.

List all agents from CLAUDE.md and .claude/agents/ in a clear table:
- Agent name
- @-mention syntax to invoke manually
- What it does (one line)
- When to use it

Then list all available slash commands from .claude/commands/ with a one-line description each.

Finally, show the recommended invocation flow:
1. Manual (@-mention): when to call a specific agent directly
2. Auto (Claude decides): when Claude delegates automatically based on context
3. Pipeline (/build-feature): when to run the full flow

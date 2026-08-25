---
name: context-engineering
description: Optimize Claude Code context for quality and total session cost.
---

# Context Engineering

Improve task quality with relevant, precise, brief context while minimizing
expected total session cost.

## Workflow

1. Inspect `CLAUDE.md` and existing Claude context files.
2. Inventory meaningful instructions without losing unique intent.
3. Classify them with `references/classify.md`.
4. Route them with `references/route.md`.
5. Propose what stays, moves, consolidates, or is removed.
6. Explain only material quality or cost effects.
7. Get approval before modifying files.
8. Apply only the approved changes.
9. Verify with `references/verify.md`.

## Rules

- Keep instructions brief, imperative, specific, and sufficient.
- Prefer targeted context over broad reads.
- Preserve or improve task quality before reducing context.
- Judge context by total downstream value, not startup size alone.
- Preserve when uncertain.

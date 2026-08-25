# Route

Route by class:

- `CORE` → `CLAUDE.md`
- `PATH` → `.claude/rules/`
- `CONTEXT` → `.claude/context/`
- `WORKFLOW` → skill
- `LOCAL` → `CLAUDE.local.md`
- `DERIVABLE` → removal candidate
- `DUPLICATE` → consolidate
- `CONFLICT` → human decision

Rules:

1. Prefer conditional loading over always-loaded context.
2. Keep `CLAUDE.md` for high-value early instructions.
3. Do not use active `@file` imports for lazy context.
4. Keep routing hints shorter than the context they route to.
5. Create only files the project needs.
6. Preserve when uncertain.

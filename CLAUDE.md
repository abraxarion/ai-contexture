# AIContexture

Builds `context-engineering` — a Claude Code skill that routes each project
instruction to the cheapest place it can still do its job. Apply the model to
this repo before shipping it.

## Context acquisition

Query `graft` first; avoid broad `grep` and large file reads. Keep context
minimal and task-relevant; avoid irrelevant, duplicate, stale, or overly broad
context.

## Writing instructions

Everything shipped here is instruction text. Write it brief, imperative,
specific, sufficient. Remove explanation before removing instruction.

## Invariants

- Preserve unique behavioral intent. Move instructions rather than delete them.
- Get approval before changing a project's context files.
- Judge context by expected total session cost, not `CLAUDE.md` size.
- Ship no manifest, metadata, telemetry, or compiler in v1.

## Where things live

- Intent: `docs/superpowers/specs/2026-08-25-context-engineering-v1-design.md`.
  Authoritative — change it before changing behavior.
- Execution: `docs/superpowers/plans/2026-08-25-context-engineering-v1-implementation-plan.md`.
- Product: `context-engineering/` — `SKILL.md` plus `references/`. It is source,
  not an installed skill; copy it under `.claude/skills/` to load it here.
- Writing under `docs/superpowers/` → read `.claude/rules/docs.md`.
- Authoring a skill → read `.claude/rules/skills.md`.
- `graft/` is a regenerable index. Never edit it by hand; run `graft build`.

## Handling CLAUDE.md

This is a living document, any agent may change it during working on this repo.
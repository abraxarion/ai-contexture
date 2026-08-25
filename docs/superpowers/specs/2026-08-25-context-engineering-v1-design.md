# Context Engineering Skill — v1 Design

**Date:** 2026-08-25  
**Status:** Design approved for review

## 1. Goal

Optimize Claude Code project context for **high task quality from relevant, precise, brief context**, while minimizing expected total session cost.

The primary objective is **better context, not merely less context**.

Optimization priority:

1. Preserve or improve task quality.
2. Maximize relevance, precision, and useful information density.
3. Minimize expected total session cost.

Total session cost includes:

- always-loaded context
- context acquisition
- cache misses
- tool calls
- latency
- retries and rework
- monetary cost

A smaller `CLAUDE.md` is useful only when it improves the total result.

## 2. Core invariants

### Preserve behavior
Do not silently lose unique behavioral requirements.

### Context quality
Prefer the smallest context set that contains the information needed for high-quality decisions.

Remove irrelevant, duplicate, stale, vague, or overly broad context before removing useful information.

Context should be:

- relevant
- precise
- brief
- non-duplicative
- authoritative
- sufficient for the task

A short always-loaded instruction may improve both quality and cost:

> Keep context minimal and task-relevant. Prefer precise targeted reads; avoid irrelevant, duplicate, stale, or overly broad context.

### Optimize total cost
A short always-loaded instruction may stay if it prevents larger downstream costs.

Example:

> Prefer Graft for repo queries; avoid broad grep and large file reads.

### Keep it simple
Use the smallest amount of structure and instruction text that reliably achieves the objective.

Generated instructions should be:

- brief
- direct
- specific
- easy to follow

### Compiler-ready
Every classification and routing decision must be representable later as structured metadata without redesigning the architecture.

v1 does **not** require a manifest.

### Preserve when uncertain
Moving information is safer than deleting it. Deletion requires high confidence.

### Idempotent
Re-running the skill should not keep rearranging an already-optimized project.

## 3. v1 Skill Layout

```text
context-engineering/
├── SKILL.md
└── references/
    ├── classify.md
    ├── route.md
    └── verify.md
```

No extra files are created unless needed.

## 4. Classification

Each meaningful instruction is classified as one of:

| Class | Meaning |
|---|---|
| `CORE` | Needed globally or before context acquisition |
| `PATH` | Relevant to predictable files or directories |
| `CONTEXT` | Relevant only to particular tasks or domains |
| `WORKFLOW` | Ordered or repeatable procedure |
| `LOCAL` | User- or machine-specific |
| `DERIVABLE` | Reliably available from authoritative repo files |
| `DUPLICATE` | Same requirement already exists elsewhere |
| `CONFLICT` | Contradicts another requirement |

Token, cache, tool, latency, and context-efficiency instructions are `CORE` when they must influence early behavior.

## 5. Routing

```text
CORE       → CLAUDE.md
PATH       → .claude/rules/
CONTEXT    → .claude/context/
WORKFLOW   → skill
LOCAL      → CLAUDE.local.md
DERIVABLE  → removal candidate
DUPLICATE  → consolidate
CONFLICT   → human decision
```

Routing rules:

1. Prefer conditional loading over always-loaded context.
2. Keep `CLAUDE.md` for high-value early instructions only.
3. Do not use active `@file` imports for lazy context.
4. Keep routing hints shorter than the context they route to.
5. When uncertain, preserve.

## 6. CLAUDE.md Admission Test

An instruction stays in `CLAUDE.md` when it has strong expected value from being available immediately.

Evaluate:

- scope
- frequency
- impact
- derivability
- routeability
- size
- effect on context acquisition

High impact includes:

- correctness
- reasoning quality
- context relevance
- precision
- architectural integrity
- safety
- cache-hit rate
- token consumption
- context pollution
- latency
- tool cost
- retry/rework cost

Early context-acquisition policy gets a strong bias toward `CORE`.

A concise context-quality instruction should also be treated as `CORE` when it guides how the agent selects and loads information.

## 7. Workflow

`SKILL.md` orchestrates:

1. Inspect `CLAUDE.md` and existing Claude context files.
2. Inventory meaningful instructions.
3. Classify with `references/classify.md`.
4. Route with `references/route.md`.
5. Present proposed changes:
   - keep
   - move
   - consolidate/remove
   - expected context benefit
6. Do not modify files before approval.
7. Apply the approved plan.
8. Verify with `references/verify.md`.

## 8. Verification

After changes, verify:

1. Every unique instruction was preserved or explicitly approved for removal.
2. Moved instructions remain discoverable when needed.
3. Context is more relevant, precise, and concise for typical tasks.
4. Always-loaded context decreased or clearly justifies its cost.
5. No unnecessary duplication, stale guidance, or conflict was introduced.
6. Re-running the skill would produce no unnecessary changes.

## 9. Generated Project Shape

The skill may produce:

```text
CLAUDE.md
.claude/
├── rules/
│   └── ...
├── context/
│   └── ...
└── skills/
    └── ...
```

Only create files and directories that the project actually needs.

## 10. Writing Standard

Generated instructions should follow:

> brief + imperative + specific + sufficient

Prefer:

> Prefer Graft for repo queries; avoid broad grep and large file reads.

Over explanatory prose.

Remove explanation before removing instruction.

## 11. Future Context Compiler

v1 deliberately keeps compiler metadata out of normal project output.

The architecture maps cleanly to future compiler stages:

```text
classify.md → classifier
route.md    → planner
verify.md   → verifier
```

Future metadata may capture:

- stable instruction ID
- source file and location
- classification
- routing decision
- confidence
- provenance
- content hash
- schema version
- verification status
- observed cost/benefit telemetry

The human-readable Markdown remains the canonical project interface.

## 12. Future Optimization Objective

The future compiler should optimize **context value and placement**, not merely file size.

Conceptually:

```text
Context Value =
    relevance
  × precision
  × usefulness
  ÷ tokens
```

This is a design principle, not a required v1 scoring formula.

Optimization priority:

```text
1. quality >= baseline, preferably improved
2. maximize task-relevant context value
3. minimize expected session cost
```

Expected session cost includes:

```text
startup context
+ context acquisition
+ cache misses
+ tool calls
+ latency
+ retries/rework
```

Subject to:

```text
behavioral requirements preserved
critical instructions preserved
required context remains discoverable
```

Telemetry may later replace heuristic estimates for context usefulness, token savings, cache behavior, tool usage, and retries.

## 13. Out of Scope for v1

- metadata manifest
- automatic telemetry collection
- compiler executable
- automatic token accounting
- complex scoring formulas
- large taxonomy of instruction types
- automatic deletion without review

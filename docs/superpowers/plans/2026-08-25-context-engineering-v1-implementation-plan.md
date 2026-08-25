# Context Engineering v1 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a minimal Claude Code `context-engineering` skill that improves task quality through relevant, precise, brief context while minimizing expected total session cost.

**Architecture:** One short orchestration file delegates three decisions to three short reference files: classify, route, verify. v1 stays metadata-free, but its categories and transformations remain stable enough to map later to compiler metadata and deterministic stages.

**Tech Stack:** Markdown, Claude Code skills/rules conventions.

**Spec:** `context-engineering-v1-design.md`

## Global Constraints

- Preserve or improve task quality before optimizing cost.
- Prefer relevant, precise, brief context over merely smaller context.
- Optimize expected total session cost, not raw `CLAUDE.md` size.
- Treat early context-acquisition and cost-control guidance as high impact.
- Keep generated instructions brief, imperative, specific, and sufficient.
- Preserve unique behavioral intent.
- Prefer moving over deleting when uncertain.
- Do not use active `@file` imports for lazy context.
- Create only files the target project actually needs.
- Keep v1 metadata-free while preserving clear future compiler seams.
- Re-running the skill should not cause unnecessary rearrangement.

---

## File Map

Create exactly:

```text
context-engineering/
├── SKILL.md
└── references/
    ├── classify.md
    ├── route.md
    └── verify.md
```

Responsibilities:

- `SKILL.md` — orchestrates inspect → classify → route → propose → approve → apply → verify.
- `references/classify.md` — defines the eight minimal instruction classes.
- `references/route.md` — maps classes to destinations and gives minimal routing rules.
- `references/verify.md` — checks preservation, context quality, cost justification, conflicts, and idempotence.

No manifest, scripts, telemetry, or compiler files in v1.

---

### Task 1: Implement the classifier contract

**Files:**
- Create: `context-engineering/references/classify.md`

**Interfaces:**
- Consumes: meaningful instructions extracted from project context files.
- Produces: exactly one primary class per instruction: `CORE`, `PATH`, `CONTEXT`, `WORKFLOW`, `LOCAL`, `DERIVABLE`, `DUPLICATE`, or `CONFLICT`.

- [ ] **Step 1: Create `classify.md` with the complete v1 classification contract**

```markdown
# Classify

Assign each meaningful instruction one primary class.

- `CORE` — needed globally or before context acquisition.
- `PATH` — relevant to predictable files or directories.
- `CONTEXT` — relevant only to a task or domain.
- `WORKFLOW` — ordered or repeatable procedure.
- `LOCAL` — user- or machine-specific.
- `DERIVABLE` — reliably available from authoritative repo files.
- `DUPLICATE` — equivalent guidance already exists.
- `CONFLICT` — contradicts another requirement.

Treat early instructions that improve context quality, cache hits, token use,
tool choice, latency, or cost as `CORE` when they must affect behavior before
additional context is loaded.

Judge by expected value, not instruction length.

When uncertain, preserve rather than delete.
```

- [ ] **Step 2: Validate classifier brevity and completeness**

Run:

```bash
grep -E 'CORE|PATH|CONTEXT|WORKFLOW|LOCAL|DERIVABLE|DUPLICATE|CONFLICT' \
  context-engineering/references/classify.md
```

Expected: all eight classes appear.

Check manually that no class has more explanation than needed to classify reliably.

- [ ] **Step 3: Validate the high-impact context rule**

Use this example mentally:

```text
Prefer Graft for repo queries; avoid broad grep and large file reads.
```

Expected classification: `CORE`.

Reason: it must influence context acquisition before repository exploration.

- [ ] **Step 4: Commit**

```bash
git add context-engineering/references/classify.md
git commit -m "feat: define context classification contract"
```

---

### Task 2: Implement deterministic routing semantics

**Files:**
- Create: `context-engineering/references/route.md`

**Interfaces:**
- Consumes: one class from `classify.md`.
- Produces: a destination or explicit human decision.

- [ ] **Step 1: Create `route.md`**

```markdown
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
```

- [ ] **Step 2: Validate one-to-one routing coverage**

Run:

```bash
grep -E 'CORE|PATH|CONTEXT|WORKFLOW|LOCAL|DERIVABLE|DUPLICATE|CONFLICT' \
  context-engineering/references/route.md
```

Expected: every classifier class has a routing outcome.

- [ ] **Step 3: Validate representative routing cases**

Expected results:

```text
"Never edit generated files."                       → CORE or PATH, depending on project scope
"Run cargo clippy after src-tauri Rust changes."   → PATH
"Architecture rationale for sidecars..."           → CONTEXT
"Release: tag, build, sign, publish."               → WORKFLOW
"MSVC is installed at C:\..."                       → LOCAL
"Project uses Rust." when Cargo.toml is authoritative → DERIVABLE
```

Do not force a route when the source instruction is ambiguous; preservation wins.

- [ ] **Step 4: Commit**

```bash
git add context-engineering/references/route.md
git commit -m "feat: define context routing rules"
```

---

### Task 3: Implement verification

**Files:**
- Create: `context-engineering/references/verify.md`

**Interfaces:**
- Consumes: original context, proposed/applied changes.
- Produces: pass/fail assessment plus any items requiring correction.

- [ ] **Step 1: Create `verify.md`**

```markdown
# Verify

After changes, check:

1. Every unique instruction was preserved or explicitly approved for removal.
2. Moved instructions remain discoverable when needed.
3. Typical task context is more relevant, precise, and brief.
4. Always-loaded context decreased or clearly justifies its cost.
5. No unnecessary duplicate, stale, or conflicting guidance was introduced.
6. Re-running the skill would make no unnecessary changes.

If any check fails, fix the routing before declaring completion.
```

- [ ] **Step 2: Validate that quality is checked before raw size**

Read the checklist.

Expected: context relevance/precision/brevity is explicitly checked, and reduced `CLAUDE.md` size is not treated as sufficient by itself.

- [ ] **Step 3: Validate deletion safety**

Test mentally:

Original:

```text
Do not replace the Rust sidecar with Node.
```

Expected: this must not be deleted merely because the repository currently uses Rust; it expresses intent not derivable state.

- [ ] **Step 4: Validate idempotence**

Scenario:

1. First run moves a Rust-only instruction to `.claude/rules/rust.md`.
2. Second run inspects the optimized project.

Expected: it leaves the already-correct instruction in place.

- [ ] **Step 5: Commit**

```bash
git add context-engineering/references/verify.md
git commit -m "feat: add context optimization verification"
```

---

### Task 4: Implement the orchestration skill

**Files:**
- Create: `context-engineering/SKILL.md`
- Read: `context-engineering/references/classify.md`
- Read: `context-engineering/references/route.md`
- Read: `context-engineering/references/verify.md`

**Interfaces:**
- Consumes: target project's `CLAUDE.md`, existing `.claude/rules/`, relevant skills, `CLAUDE.local.md`, and context files when present.
- Produces: an approved, minimal context-routing change set and verification result.

- [ ] **Step 1: Create `SKILL.md`**

```markdown
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
```

- [ ] **Step 2: Check that the orchestrator does not duplicate reference content**

Compare `SKILL.md` with the three reference files.

Expected:

- `SKILL.md` names the workflow.
- `classify.md` owns taxonomy.
- `route.md` owns destinations.
- `verify.md` owns post-change checks.
- Detailed category definitions are not copied into `SKILL.md`.

- [ ] **Step 3: Check instruction length**

Run:

```bash
wc -l context-engineering/SKILL.md \
      context-engineering/references/classify.md \
      context-engineering/references/route.md \
      context-engineering/references/verify.md
```

Expected: each file remains small enough to scan quickly. Do not add prose merely to hit or avoid an arbitrary line count.

- [ ] **Step 4: Check for accidental eager imports**

Run:

```bash
grep -RInE '(^|[[:space:]])@[^[:space:]]+\.md' context-engineering/ || true
```

Expected: no active Markdown imports used as lazy-routing references.

- [ ] **Step 5: Commit**

```bash
git add context-engineering/
git commit -m "feat: add context-engineering skill"
```

---

### Task 5: Run acceptance scenarios

**Files:**
- No persistent files required.

**Interfaces:**
- Consumes: completed four-file skill.
- Produces: evidence that v1 meets the design.

- [ ] **Step 1: Test a bloated `CLAUDE.md` scenario**

Use a temporary project containing:

```markdown
# CLAUDE.md

Prefer Graft for repo queries; avoid broad grep and large file reads.
Run cargo clippy after changing Rust code.
The release process is: update changelog, tag, build, sign, publish.
The project uses Rust.
Do not replace the Rust sidecar with Node.
Architecture details: ...
```

Expected proposal:

```text
CORE       Graft query policy
PATH       cargo clippy rule
WORKFLOW   release process
DERIVABLE  "project uses Rust" if Cargo.toml confirms it
CORE/CONTEXT preserve "Do not replace Rust sidecar with Node" based on scope
CONTEXT    architecture details
```

- [ ] **Step 2: Check the quality outcome**

Expected:

- the normal session starts with less irrelevant detail;
- high-impact context-acquisition guidance remains immediately available;
- task-specific knowledge remains discoverable;
- no unique intent is silently removed.

- [ ] **Step 3: Check the cost outcome**

Expected:

- always-loaded context is reduced unless retained instructions have clear downstream value;
- broad repository reads are discouraged when targeted queries suffice;
- no eager `@file` routing defeats lazy loading.

- [ ] **Step 4: Check idempotence**

Run the skill conceptually against the already-optimized result.

Expected: no unnecessary moves, rewrites, file splits, or wording churn.

- [ ] **Step 5: Final repository check**

Run:

```bash
find context-engineering -maxdepth 2 -type f -print | sort
```

Expected:

```text
context-engineering/SKILL.md
context-engineering/references/classify.md
context-engineering/references/route.md
context-engineering/references/verify.md
```

No manifest, compiler, telemetry, or unrelated files.

- [ ] **Step 6: Commit acceptance adjustments only if needed**

```bash
git add context-engineering/
git commit -m "fix: tighten context-engineering acceptance behavior"
```

Skip this commit when acceptance requires no changes.

---

## Completion Criteria

v1 is complete when:

- the four intended files exist;
- classification and routing cover all eight classes;
- quality is explicitly prioritized over mere context reduction;
- total-session-cost guidance includes context acquisition and cache/token effects;
- generated instructions are brief and actionable;
- unique intent is preserved;
- active `@file` imports are not used for lazy routing;
- the skill requires approval before mutation;
- acceptance scenarios pass;
- a second run would cause no unnecessary churn;
- no metadata/compiler machinery has been introduced.

## Future Compiler Seam

Do not implement this in v1.

The later mapping is intentionally direct:

```text
classify.md → classifier
route.md    → planner
verify.md   → verifier
SKILL.md    → orchestration contract
```

Future metadata can add stable IDs, provenance, source locations, confidence,
hashes, schema versions, verification state, and observed cost/quality
telemetry without changing the v1 human-facing routing model.

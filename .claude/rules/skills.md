# Skills

Applies to skills authored here, including `skills/ai-contexture/`.

- Lay out a skill as `SKILL.md` plus `references/*.md`. Add nothing else.
- Give frontmatter `name` and `description`. Make the description one line
  stating when to use the skill. Add `context: fork` and `background: false`
  only when the skill must run in an isolated subagent.
- Let `SKILL.md` name the workflow steps and delegate every definition to a
  reference file. Do not copy reference content into it.
- Give each reference file one decision: `classify.md` owns the taxonomy,
  `route.md` owns destinations, `verify.md` owns post-change checks.
- Point at reference files by relative path (`references/route.md`). Never use
  `@file` imports — they load eagerly and defeat lazy routing.
- Keep every file short enough to scan. Do not pad to reach a line count.
- Require approval before the skill modifies any file.

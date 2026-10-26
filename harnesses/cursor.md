# Harness: Cursor

Wire this when the user chooses `cursor` (or `both`) in `/setup-project`.

## Project files

- `AGENTS.md` — committed team/solo contract. Cursor loads it as the
  project agent instructions.
- `.cursor/skills/handoff/SKILL.md` — copy of `skills/handoff/SKILL.md`
  with HISTORY/TESTING lines filled or removed.
- `.gitignore` includes `.cursor/rules/local.mdc`.

Do **not** create `.claude/`, `CLAUDE.md`, or Claude SessionStart hooks
unless the user also chose `claude`.

## Compact

Happy path: `/handoff`, then a **fresh chat** with the pasted block.
Do not compact-and-stay after handoff — the old chat is leftover.

Cursor has no SessionStart hook that re-injects files after summarize.
If Cursor auto-summarizes mid-task (or the user ran `/summarize` by
accident), the agent re-reads PROJECT.md and AGENTS.md from disk (the
re-read line in the project AGENTS.md). Run `/handoff` *before* any compact, without asking, so the delta is not lost (project work only).

## Personal rules in a shared repo

Create `.cursor/rules/local.mdc` only when the user asks or when splitting
a shared repo. Gitignored. Frontmatter:

```yaml
---
description: Personal collaboration rules for this repo (not team)
alwaysApply: true
---
```

## User-level (once per machine)

Skills (junction or copy):

```
<methodology-root>/skills/setup-project    →  ~/.cursor/skills/setup-project
<methodology-root>/skills/update-project   →  ~/.cursor/skills/update-project
```

`handoff` is project-local after setup. A global copy is optional.
`sync-upstream` is not installed globally. It is a project skill in this
repo's `.claude/skills/` (Cursor reads that folder too) and only applies here.

## Optional preferences

Do **not** install `~/.cursor/rules/agentic-coding.mdc` by default. An
`alwaysApply: true` user rule is read by every Cursor agent, including repos
that do not use this methodology.

- Every repo: user-level `~/.cursor/rules/agentic-coding.mdc`, pointing at
  `global/AGENTS.md`, with machine-private facts below. Template:
  `global/cursor.rule.mdc`.
- Selected repos only (recommended): put that pointer and the private facts
  in the repo's gitignored `.cursor/rules/local.mdc`.

## Module rules

Not nested AGENTS.md. A small `.cursor/rules/<area>.mdc` with `globs`
and local behavior plus pointers into `docs/agentcontext/` — never a
knowledge dump.

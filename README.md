# agentic-coding-methodology

My methodology for working with coding agents on projects that run for months
and thousands of chat messages. It solves one problem: in week twelve, neither
I nor the agent remembers why something was decided in week three.

Core idea: context is the scarce resource. After every task, the relevant
knowledge is written to files in the repo — each with a defined load behavior
(always / targeted / on demand). Every fresh session reads those files and
continues where the last one stopped. Forgetting is fine; losing the *why* is not.

**The full explanation, with diagrams:**
[henroxx.github.io/agentic-coding-methodology](https://henroxx.github.io/agentic-coding-methodology/)

## Harnesses

`AGENTS.md` is the tool-agnostic contract and `docs/agentcontext/` is plain
Markdown — the methodology itself doesn't care which agent runs it. A harness
is only the wiring that makes a given tool load those files: Cursor reads
`AGENTS.md` directly; Claude Code gets a thin `CLAUDE.md` that `@`-imports
`AGENTS.md` plus a SessionStart hook, so nothing is written twice. `/setup-project` asks which
harness to wire — there is no default. See `harnesses/`: one small file per
harness, and adding another (e.g. codex) is one more.

## Quick start

1. Make the skills available globally (once), for your harness.

   Cursor (macOS/Linux):

   ```bash
   ln -s <clone>/skills/setup-project    ~/.cursor/skills/setup-project
   ln -s <clone>/skills/update-project   ~/.cursor/skills/update-project
   ```

   Cursor (Windows PowerShell; junctions do not require Developer Mode):

   ```powershell
   $clone = "C:\path\to\agentic-coding-methodology"
   New-Item -ItemType Junction -Path "$HOME\.cursor\skills\setup-project" -Target "$clone\skills\setup-project"
   New-Item -ItemType Junction -Path "$HOME\.cursor\skills\update-project" -Target "$clone\skills\update-project"
   ```

   Claude Code (macOS/Linux):

   ```bash
   mkdir -p ~/.claude/skills
   ln -s <clone>/skills/setup-project    ~/.claude/skills/setup-project
   ln -s <clone>/skills/update-project   ~/.claude/skills/update-project
   ```

   Claude Code (Windows PowerShell): the same `New-Item -ItemType Junction`
   lines as above, with `$HOME\.claude\skills\...` as the path.

   Claude Code re-injects `PROJECT.md` after a compact via a SessionStart
   hook that `/setup-project` adds to each repo.

   `/sync-upstream` (pull Henroxx into this fork) is a project skill in
   `.claude/skills/` of this repo. It is never installed globally.

   Optional: wire `global/AGENTS.md` only if you want its preferences in
   **every** repo. Cursor user rules with `alwaysApply: true` affect every
   Cursor agent, including repos that do not use this methodology. Template:
   `global/cursor.rule.mdc`. For selected repos only, put the same pointer and
   your private machine facts in that repo's gitignored
   `.cursor/rules/local.mdc`. Claude Code can use an `@`-import in
   `~/.claude/CLAUDE.md`.

2. In any new project, run:

   ```
   /setup-project
   ```

   It asks which harness to wire (cursor / claude / both) plus two short
   question rounds, then scaffolds `AGENTS.md`, `docs/agentcontext/`, and the
   `/handoff` skill from the templates in this repo — as a single revertible
   commit.

## Layout

```
AGENTS.md                # this repo's own working rules
METHODOLOGY.md           # the concept: workflow, file roles, principles
CLAUDE/                  # read-only vendor copy of Henroxx's tree — do not edit
global/                  # user-level preferences
  AGENTS.md              # shared, generic baseline (name/machine facts stay local)
  cursor.rule.mdc        # alwaysApply wrapper for ~/.cursor/rules/
templates/               # source of truth for scaffolded files
  AGENTS.project.md      # project-level AGENTS.md template
  agentcontext/          # PROJECT, DETAILS, HISTORY, TESTING templates
skills/
  setup-project/         # scaffolds the methodology into a repo
  update-project/        # pulls methodology updates into a scaffolded repo
  handoff/               # ends a session cleanly, persists the session delta
.claude/skills/
  sync-upstream/         # this repo only: translate Henroxx updates into this fork
harnesses/               # per-tool wiring (contract file, skills, local rules)
  README.md              # the harness contract + mapping table
  cursor.md              # Cursor wiring
  claude.md              # Claude Code wiring
  upstream.md            # Henroxx path map, kept-local list, upstream stamp
variants/                # deltas on the core: work (license rules), thesis (planned)
```

The `docs/agentcontext/` file structure is plain Markdown and tool-agnostic;
only the thin harness wrapper differs per tool.

## Status

Generic, harness-neutral core (Cursor and Claude Code), plus the work variant's
license rules. Remaining work deltas (team conventions) and the thesis variant
are planned — see `variants/README.md`.

# Harnesses

The knowledge files (`docs/agentcontext/`) and `AGENTS.md` are the core, and
they are tool-agnostic. A harness is only the wiring that makes a specific
IDE/agent load those files.

`/setup-project` **asks** which harness to wire (`cursor`, `claude`, or
`both`) — there is no silent default, because on a fresh repo there is nothing
to detect yet. On an already-scaffolded repo, `/update-project` and `/handoff`
read what exists on disk (`.cursor/skills/` vs `.claude/skills/`, the
`AGENTS.md`/`CLAUDE.md` stamp) and infer the harness without re-asking. Never
scaffold a harness the user did not choose.

| Harness | Contract file | Skills | Local (gitignored) |
|---|---|---|---|
| cursor | `AGENTS.md` | `.cursor/skills/` | `.cursor/rules/local.mdc` |
| claude | `AGENTS.md` plus a thin `CLAUDE.md` that `@`-imports it | `.claude/skills/` | `CLAUDE.local.md`, `.claude/settings.local.json` |

`AGENTS.md` is the single source of truth in every case. Claude Code reads
`CLAUDE.md` natively, so the claude harness keeps a short pointer there whose
`@AGENTS.md` line imports the contract instead of a second copy of the rules —
nothing is written twice.

Module-specific behavior:

- Cursor: `.cursor/rules/<area>.mdc` with `globs` (e.g. `backend/**`)
- Claude Code: optional `backend/CLAUDE.md` — only if that harness is on

Adding a harness (e.g. codex): drop one `<name>.md` here that names its
contract file, its skills path and its local-rules path, then teach
`/setup-project` to offer it. One small file per harness, nothing more.

Henroxx's Claude-only tree is vendored under `CLAUDE/` at the repo root.
How to pull his updates without clobbering this fork: `upstream.md`.

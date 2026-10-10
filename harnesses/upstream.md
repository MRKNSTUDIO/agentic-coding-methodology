# Upstream (Henroxx)

This fork keeps Henroxx's Claude-only tree as a **read-only vendor copy**
under `CLAUDE/`. Never edit those files except by replacing the folder with a
newer verified upstream `main`. Fork-owned files live on paths he does not
use; merging upstream `main` onto this branch would fight those paths and can
wipe local additions. Use `/sync-upstream`, not a merge.

## Remote resolution

Canonical upstream:
`https://github.com/Henroxx/agentic-coding-methodology.git`.

A fresh clone normally uses `origin` for its own fork. The recommended layout
is therefore:

- `origin` — your fork
- `upstream` — the canonical Henroxx URL above

Existing clones may use different names. `/sync-upstream` resolves the remote
by its normalized fetch URL and verifies it before fetching; it never assumes
that `origin` means Henroxx. If no matching remote exists:

```powershell
git remote add upstream https://github.com/Henroxx/agentic-coding-methodology.git
```

## Path map

Translate his paths onto fork-owned files. Claude-only wiring stays in
`harnesses/claude.md` or is ignored.

| Henroxx | This fork |
|---|---|
| `CLAUDE/templates/CLAUDE.project.md` | `templates/AGENTS.project.md` |
| `CLAUDE/templates/agentcontext/*` | `templates/agentcontext/*` |
| `CLAUDE/skills/<name>/` | `skills/<name>/` |
| `CLAUDE/variants/*` | `variants/*` |
| `CLAUDE/METHODOLOGY.md` | `METHODOLOGY.md` |
| `CLAUDE/global/CLAUDE.md` | `global/AGENTS.md` (generic parts only; name and machine facts stay local) |
| SessionStart compact hook, `CLAUDE.md` as the contract, `~/.claude/` paths | `harnesses/claude.md` only |
| His root `CLAUDE.md` (Henry-centric working rules) | propose into this repo's `AGENTS.md`; do not replace the thin pointer `CLAUDE.md` |

## Kept local — never replace with his version

- Root `README.md` — Cursor / harness story
- Root `CLAUDE.md` — thin pointer at `AGENTS.md`
- `docs/` — his GitHub Pages site; leave it unless the site is forked on purpose
- **Cursor continue loop** — after `/handoff`, a fresh chat and the pasted
  block. Do not write his "compact and stay" path back into
  `METHODOLOGY.md`, `harnesses/cursor.md`, `AGENTS.md`, or
  `templates/AGENTS.project.md`. Claude may keep compact-and-continue;
  that harness has a SessionStart hook. Reason: after handoff the old chat
  is leftover; compacting it adds nothing, and compacting *before* handoff
  can drop the delta.

## Stamps

- `vendor:` below — last upstream `main` copied verbatim into `CLAUDE/`.
- `translated:` below — last upstream commit whose applicable changes are
  either translated into this fork or recorded above as deliberately kept
  local. Declining a proposal temporarily does not advance this stamp, so the
  delta appears again next time.
- `methodology: <hash> (<date>)` at the bottom of scaffolded `AGENTS.md` —
  this clone's HEAD, for consumer `/update-project`. Do not mix the two.

vendor: 3266dfe (2026-08-26)
translated: 3266dfe (2026-08-26)

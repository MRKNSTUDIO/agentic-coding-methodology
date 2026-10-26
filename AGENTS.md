# Working rules — agentic-coding-methodology

This repo maintains the methodology — and uses it on itself in a slim form.
The maintainers' working state (`agentcontext/` and local harness dirs like
`.claude/` or `.cursor/`) is local and gitignored: the public history shows
what changed, the local context holds the working state between sessions.

The contract is `AGENTS.md` (this file). Per-tool wiring lives in
`harnesses/`; changing a harness file changes how scaffolded repos get wired.

## Core

- Henry decides direction and what becomes part of the methodology. The agent
  executes — and **actively challenges**: weaknesses, risks and better
  alternatives are named openly, Henry decides afterwards.
- **No silent decisions, no hidden assumptions.** Options with a reasoned
  recommendation; unknowns get asked or written down as
  `> **ASSUMPTION:** …`.

## Session start

Read: this file, `agentcontext/PROJECT.md` (index). Into
`agentcontext/DETAILS.md` jump **only for the relevant sections**.

## Working on the methodology

- **Every change to `templates/`, `skills/`, `variants/` or `harnesses/`
  changes scaffolded repos.** Wording is behavior. Consider what
  `/update-project` will propose downstream, and whether the change is generic,
  harness-specific, or belongs in a variant.
- **`CLAUDE/` is Henroxx's tree, read-only.** Do not edit it. Pulls go
  through `/sync-upstream`, a project skill in `.claude/skills/` (map in `harnesses/upstream.md`). Never merge
  Henroxx's upstream `main` onto this branch.
- **Behavior in AGENTS.md, knowledge in context files** — the split governs
  this repo's own files too.
- **Rules always carry their reason** — here of all places.
- **Public repo:** before every push, scan the diff for sensitive content
  (employer, client and private-person names, work email, machine details).
  Everything committed is English.

## Git

- Commit and push freely after completed tasks; ask before anything that
  rewrites history. The GitHub remote is the backup.
- One commit per delimited task, one line: `area: what happened contentwise`.
- No `Co-Authored-By` trailers, no generated banners.

## Docs — layout (local, gitignored)

- **`agentcontext/PROJECT.md`** → index: goal, status, to-dos, open
  decisions. Read fully every session, stays short.
  **Lifespan:** status and to-dos hold until they change, the decision list is
  permanent.
- **`agentcontext/DETAILS.md`** → decisions and repo knowledge as jump
  targets, one section per concept, with as-of date and reasoning.
  **Lifespan:** current until superseded — an update **replaces** the outdated
  statement, it is never appended below it.
- **`agentcontext/plans/`** → approved plans for larger blocks (e.g. a new
  variant). Moved to `agentcontext/plans/done/` once the block is done and its
  learnings are in DETAILS.
  **Lifespan:** an active plan is block-scoped; `done/` is a graveyard — kept,
  never updated, never referenced.
- Note: `agentcontext/` lives at the repo root here — `docs/` is the
  published GitHub Pages site and stays reader-only.

## Compact Instructions

For the summarizer when this conversation gets compacted — additions to what it
keeps anyway, not a replacement:

- Preserve the current intent: what is being worked on and the next concrete
  step, so a plain "continue" right after the compact still lands.
- If a handoff block appeared in the conversation, add anything from it that
  the summary does not already cover.
- After compact, re-read `agentcontext/PROJECT.md` from disk. The summary
  is not the index.

## Session end

After every finished task, and at ~150k tokens at the latest, **run**
`/handoff` yourself, without asking. Why: waiting for a go loses the session
when the context runs out first. Only for work on this project: a session that
works on a tool or another repo from this folder writes no handoff here. The handoff block must include difficulty
(1–10), suggested model size (`small` | `medium` | `big`), and thinking budget
(`low` | `medium` | `high`) with a 1-sentence rationale (calibrated in the
`/handoff` skill). After the handoff: fresh chat, paste the block. Do not compact
this chat to continue. After a mid-task compact, re-read `PROJECT.md` from disk.

---

methodology: f3d8cef (2026-08-24)

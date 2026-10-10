---
name: setup-project
description: Scaffolds the agentic-coding methodology into the current repo — AGENTS.md, docs/agentcontext/, and the /handoff skill. Asks which harness to wire (cursor / claude / both). Use when the user asks to set up the methodology / working setup in a repo. Asks two short question rounds, then sets everything up as a single revertible commit.
disable-model-invocation: true
argument-hint: "free text: harness (cursor/claude/both), variant (work/thesis), and/or project info already specified"
---

# Set up the methodology in this repo

Templates are the source of truth. They live in the methodology repo.

Resolve **methodology root** (first match):

1. Environment variable `AGENTIC_METHODOLOGY_ROOT`
2. The resolved target of this installed skill's symlink/junction: from
   `skills/setup-project/SKILL.md`, go up three levels and verify that the
   result contains `templates/AGENTS.project.md`. Claude Code names the
   skill's base directory when it loads — resolve that path with `realpath`
   (or `readlink -f`); Cursor: resolve `~/.cursor/skills/setup-project`
3. The current repo, if it contains `templates/AGENTS.project.md` and
   `skills/setup-project/SKILL.md`

If the skill was copied rather than linked and none match, stop and ask the
user to set `AGENTIC_METHODOLOGY_ROOT` to their clone. Never guess a
machine-specific path.

```
<methodology-root>/templates/
<methodology-root>/harnesses/
<methodology-root>/variants/
```

Copy templates and replace `{{PLACEHOLDERS}}` — never invent the files from
scratch.

The invocation may carry free text: a **harness** (`cursor`, `claude`,
`both`), a variant name (`work`, `thesis`), and/or project info. How each is
handled:

- Harness: **no default — this is a required question** (see round 1). A fresh
  repo has no `.cursor/`/`.claude/` to detect, so the choice is always asked
  and confirmed. If the invocation already named one, that answer stands and
  the question is skipped. Never scaffold Claude Code files unless the user
  chose `claude` or `both`, and never scaffold Cursor files unless they chose
  `cursor` or `both`.
- If a variant is named, read `<methodology-root>/variants/<variant>.md` and
  apply its deltas; if that file doesn't exist yet, say so and continue with
  the generic setup.
- Everything else in the free text counts as answers to the question protocol
  below — never re-ask what was already provided.

Once the harness is settled, read `<methodology-root>/harnesses/<name>.md`
(for `both`, read `cursor.md` then `claude.md`) and apply that wiring on top
of the generic files.

## Rules for this setup run

- The user knows git and coding agents. No basics. Per created file exactly
  one sentence: what it is for and when it is read.
- Don't show file contents in the chat. Create, then report briefly what
  exists now.
- Questions as normal chat prose, not selection forms — in Claude Code that
  means no `AskUserQuestion`. Always with a recommendation and short
  reasoning, so a "yes, do it" suffices.
- Follow the question protocol below: one round of questions, one proposal
  block, one assumptions list. Everything else you decide yourself with a
  stated default. One extra question only if a real ambiguity changes the
  structure — never for cosmetics.
- Visibility instead of questions: where you decide yourself, the decision is
  named in the final report, together with the sentence that overturns it.
- Nothing irreversible. Check `git status` before the first change; if
  uncommitted changes exist, say so and ask whether to secure them first. The
  whole setup becomes one commit, so one command removes it.
- If an `AGENTS.md`, `CLAUDE.md`, or `docs/agentcontext/` already exists:
  don't overwrite. Read what's there, propose a merge, ask.
- `AGENTS.md` carries **behavior rules only** — never project knowledge.
  Beyond the template's slots, nothing about stack, architecture or repo
  status goes in; such facts belong in PROJECT.md/DETAILS.md.

## Question protocol

### Round 1 — what only the user knows

1. **Language for the docs.** Recommendation: everything English (files travel
   across tools and teammates; the chat mirrors whatever language the user
   writes anyway). Alternative: docs in the user's language, code and git
   English.
2. **Topic, scope and constraints in three sentences.** Follow up exactly
   once if the delimitation is missing: what explicitly does *not* belong to
   the project? Don't interrogate further — resolve the rest via marked
   assumptions.

3. **Harness — a clear choice, no default.** Ask which harness to wire:
   `cursor` (AGENTS.md + `.cursor/skills`), `claude` (adds a thin `CLAUDE.md`
   pointer + `.claude/` skills and hook), or `both`. Skip only if the
   invocation already named one. If the repo already contains `.cursor/` or
   `.claude/`, name that as the recommendation — but still surface the choice,
   never decide it silently.

### Round 2 — one proposal block, not four questions

Present everything else as a finished proposal in one message: four numbered
lines, each with the chosen option and half a sentence of why. Close with:
"Does this fit? Otherwise just tell me the numbers that should change."

| # | Decision | Default to propose |
|---|----------|--------------------|
| 1 | Optional files | HISTORY.md and TESTING.md both on. HISTORY is later the raw material for any write-up; TESTING prevents nobody knowing after twenty sessions what was verified. |
| 2 | Explanation depth | Explain compactly, agent writes — concepts in two or three sentences upfront, code after. Alternatives: everything explained before code · only genuinely new things. |
| 3 | Git permissions | Commit and push freely, PR and merge only after asking. Alternatives: ask before every git action · everything free. |
| 4 | Testing | Throwaway sanity checks after every step, few real tests for the critical parts. Alternative: full test suite from the start — right when testability is itself part of the topic. |

If the user says "just use your defaults": skip round 2, take all defaults,
continue building — but list the four settings in the final report and note
that one sentence in the chat changes any of them.

### Round 3 — assumptions for review

After building: list the assumptions that landed in PROJECT.md, as a short
list with "correct?". No question forms, no repetition of what the user
already said.

## Setup steps

1. **Survey the repo, then round 1.** Without asking: git repo? which branch?
   clean tree? language/toolchain? existing AGENTS.md, CLAUDE.md,
   docs/agentcontext/, .cursor/, .claude/, README? If code exists, skim it for
   orientation — no full audit. Then ask round 1 and wait.

2. **Present round 2** and wait. On "just do it": continue with defaults.

3. **Scaffold the core** (every harness):

   ```
   AGENTS.md                         <- templates/AGENTS.project.md
   docs/agentcontext/PROJECT.md      <- templates/agentcontext/PROJECT.md
   docs/agentcontext/DETAILS.md      <- templates/agentcontext/DETAILS.md
   docs/agentcontext/plans/          (empty directory, tracked via .gitkeep)
   docs/agentcontext/plans/done/     (empty directory, tracked via .gitkeep)
   ```

   Then apply the chosen harness file(s):

   - **cursor**: `.cursor/skills/handoff/SKILL.md` from
     `skills/handoff/SKILL.md`. No Claude Code hook — Cursor has no
     SessionStart equivalent. Compact recovery in the project is the
     re-read line in AGENTS.md.
   - **claude**: per `harnesses/claude.md` — `CLAUDE.md` pointer that
     `@`-imports `AGENTS.md`, `.claude/skills/handoff/SKILL.md` from
     `skills/handoff/SKILL.md` (same filling rules as below), and the
     SessionStart compact hook merged into `.claude/settings.json`.
   - **both**: cursor first, then claude extras.

   In the handoff skill, fill `{{HISTORY_LINE}}` / `{{TESTING_LINE}}` only if
   the file exists per round 2, otherwise remove the line entirely. HISTORY
   line template: "Completed work steps chronologically (date | what | why) →
   `docs/agentcontext/HISTORY.md`." TESTING line template: "New or changed test
   scenarios → `docs/agentcontext/TESTING.md`."

4. **Actually fill PROJECT.md.** No empty skeleton. Write Goal & Scope,
   Status and a first to-do block from round 1 and the repo survey. Where
   information is missing, write a visibly marked assumption instead of
   leaving a gap — uniformly as `> **ASSUMPTION:** …`. Assumptions can be
   corrected; gaps get overlooked. Exactly these lines are round 3.

   In DETAILS.md create the first two or three section heads that follow from
   the topic, each with an as-of date. Empty sections are fine — they are the
   landing zone.

5. **Optional files** per round 2, from `templates/agentcontext/`.

6. **Write AGENTS.md.** Replace all slots (table below). Delete unneeded
   lines entirely — no empty placeholders left standing. Every rule keeps its
   reasoning unless self-explanatory.

7. **Clean up and secure as one commit.** Check/extend `.gitignore`
   (`.DS_Store`, venv, caches, and per harness the personal files —
   cursor: `.cursor/rules/local.mdc`; claude: `CLAUDE.local.md`,
   `.claude/settings.local.json`. Personal rules never get committed).
   Version `docs/agentcontext/`, `AGENTS.md` and the harness wiring
   (`.cursor/skills/`, or `CLAUDE.md`, `.claude/skills/` and
   `.claude/settings.json`) — the project knowledge travels with the code and
   lands in the backup. Then one
   commit for the whole setup, respecting the git permission from round 2.
   Note the commit hash for the final report.

8. **Final report — round 3.** Briefly in the chat, in this order: created
   files (one line each) · harness chosen · the assumptions from PROJECT.md
   with "correct?" · what you decided yourself without asking (half a
   sentence each) · the revert command verbatim (`git revert <hash>`, or
   `git reset --hard HEAD~1` if not pushed) · the note that `/handoff` is now
   available (~150k tokens and after every finished task) · a proposal for
   the first concrete work step. Then stop and wait.

## Slot reference

All placeholders in `templates/AGENTS.project.md`. What doesn't come from a
question, you decide yourself.

| Placeholder | Source | Note |
|---|---|---|
| `{{PROJECT}}`, `{{NAME}}` | repo / git config | Project name from repo name or topic; name from git config. |
| `{{CORE_SENTENCE}}` | you, from round 2 | One sentence fixing the division of labor. Not generic — it should reflect the user's answers. |
| `{{ADDITIONAL_START_FILES}}` | round 2 · 1 | Only add what really must be read at session start. HISTORY does not belong here. |
| `{{QUESTION_STYLE}}` | you | Sharpen or remove entirely if the base rule suffices. |
| `{{TESTING_PHILOSOPHY}}` | round 2 · 4 | Concrete: which kind of check, when, recorded how. |
| `{{CODE_LANGUAGE}}` | round 1 | — |
| `{{DEPENDENCY_RULES}}` | repo | Name the project's package manager (uv add, pip, npm …). Tooling behavior only — the tech stack itself is project knowledge and belongs in DETAILS, not in AGENTS.md. |
| `{{GIT_PERMISSIONS}}` | round 2 · 3 | Spell out explicitly what is allowed without asking and what isn't. |
| `{{BACKUP_NOTE}}` | you | If no cloud sync exists: write "git is the only backup". It changes behavior noticeably. |
| `{{OPTIONAL_DOC_LINES}}` | round 2 · 1 | One line per chosen file, in the style of the existing docs list. |
| `{{METHODOLOGY_VERSION}}` | methodology repo | `git -C <methodology-root> rev-parse --short HEAD` plus date, e.g. `f359a63 (2026-08-20)`. Lets a later update run see what this repo was scaffolded from. |

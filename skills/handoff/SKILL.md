---
name: handoff
description: Ends a session cleanly — persists the session delta to PROJECT/DETAILS{{/HISTORY}}, removes what the finished work block leaves behind, and generates a copy-paste handoff block for the next session. Run it without asking before a compact (guideline ~150k tokens) and after a finished task only when the next task is settled (approved plan, long-running block), and only when the session worked on this project, not on a tool or another repo; also when the user says /handoff, "end the session" or "hand over". After the block: new chat (Claude Code: /clear), paste it — do not compact this chat.
argument-hint: "What will the next session work on?"
---

# Session handoff

Goal: a fresh session (new agent, small context) can continue seamlessly.
Nothing that only exists in the chat gets lost, nothing gets documented twice,
and nothing outdated is left standing.

## Step 1 — Delta check

Go through the current session and collect what was decided, learned or
changed but is not persisted anywhere yet. Mapping:

- Decisions (architecture, methodology, deliberate deviations) →
  `docs/agentcontext/DETAILS.md`, matching section, a few sentences **with reasoning**
- Status changes, finished and new to-dos, resolved open questions →
  `docs/agentcontext/PROJECT.md`
- Progress against an active plan → update `docs/agentcontext/plans/<topic>.md`;
  if the block is done, fold its learnings into DETAILS and move the plan to
  `docs/agentcontext/plans/done/`
{{HISTORY_LINE}}
{{TESTING_LINE}}
- New behavior rules for the collaboration → `AGENTS.md`

Only the **delta of this session** — no full audit of all files.

**Verify the index before writing to it.** Check PROJECT.md's status against
the actual state (git log, branch, what exists on disk) instead of trusting it.
A wrong status is the one error a handoff carries into a fresh session, where
it can no longer be recognized as wrong.

## Step 2 — Persist

Write the delta items to their places. Then list transparently what was added
where. Only ask when a mapping is genuinely unclear. If there is nothing to
persist: say so explicitly.

## Step 3 — Collect the garbage

Writing is the easy half. Deletion only happens if it happens here, so run
these checks — briefly, not as an audit:

- **Finished plans:** learnings folded into DETAILS, then the plan moved to
  `plans/done/`. The move follows the fold, it never replaces it.
- **Block-scoped leftovers:** progress notes, sanity-check records, and
  `> **TEMPORARY.**` blocks whose condition is met — deleted.
- **Appends that should have been replacements:** did a DETAILS update replace
  the outdated statement, or is it now stacked below it? Replace it.
- **Duplicated facts:** the same fact in several files — keep the one with the
  longer lifespan, reference it from the others.

If this project keeps a history log: the new entry is at most 3 lines and does
not repeat what DETAILS already holds. Finding nothing to clean up is a valid
outcome — say so in one line instead of inventing work.

## Step 4 — Handoff block

Always output the block. The next step is a **fresh chat** — in Cursor a
new chat, in Claude Code `/clear` (or a new session) — and the block is its
first message. Do not tell the user to
`/summarize` or `/compact` this chat; after handoff the old transcript is
leftover.

Introduce it with one sentence: the text below is the first message of a
new chat.

Then a Markdown code block, copy-paste-ready. Include **only the parts that
apply**:

- **Task** — what the next session works on: from the argument, or the settled
  next step of an approved plan or long-running block. Never guess it from the
  to-dos. Not settled: stop before the block, name what is open with one
  recommendation, and write the block once the user has picked
- **Difficulty** — score 1–10 | Model: `small` | `medium` | `big` | Thinking: `low` | `medium` | `high` (one-sentence rationale). Calibration:
  - 1–3: `small` model, `low` thinking — routine edits, docs/copy changes, isolated bugfixes, small self-contained helpers.
  - 4–6: `medium` model, `medium` thinking — multi-step features, component extractions, standard logic with well-bounded interfaces.
  - 7–8: `big` model, `high` thinking — complex architectural pivots, async lifecycle/concurrency, tricky state machines, cross-system integration.
  - 9–10: `big` model, `high` thinking — high ambiguity, cross-process OS edge cases, deep distributed debugging or multi-subsystem overhaul.
- **State** — branch, last commit, what works, what is open
- **Reading pointers** — references only (e.g. "DETAILS.md → section
  Sessionization"), never duplicate content that already lives in files
- **Session patterns** — working patterns from this session that are not
  file-worthy but will make the next session faster. Anything that will matter
  again belongs in DETAILS instead. Nothing non-obvious to report? Leave the
  section out — never invent one to fill a slot.
- **First action** — the concrete first step or first question

Rules for the block:

- Readable standalone, without this chat history
- No secrets, API keys or sensitive data
- At most ~50 lines, and **shorter is better.** A block that says little more
  than "read these two files, then do X" is the good case: it means the files
  are carrying the state. Don't pad it.

Close with one line: new chat (Claude Code: `/clear`), paste the block. Then
stop.

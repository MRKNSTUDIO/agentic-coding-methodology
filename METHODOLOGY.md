# The Methodology

For projects that run over months. Grown practice from several long projects,
not a finished recipe — the files are living documents, including the ones
containing the rules.

## The problem

Two effects, both well known: a full context window makes access to any single
item in it fuzzier. And summarizing (compaction, memory extraction) keeps
*what* was decided but loses *why* — exactly the part you need two months later.

So this methodology does not rely on the agent remembering. Knowledge is
deliberately written out; then every session is allowed to forget.

## The loop

One task, one cycle. At the end, the knowledge is in files instead of the chat:

```
Plan → your OK → Task (persist decisions the moment they fall)
   → /handoff
   → fresh chat, paste the handoff block
   → next task
```

- **Plan.** Anything bigger than a small change gets a plan and an OK first —
  then code. Approved plans for larger work blocks are written to
  `docs/agentcontext/plans/`, so later sessions can verify against the plan, not just
  against to-dos.
- **Task.** Decisions go into the files the moment they are made, not at the
  end. What only exists in the chat is lost.
- **/handoff.** After every finished task, at latest around ~150k tokens.
  Collects what only lived in the chat, writes it to `docs/agentcontext/`, and outputs a
  short block for the next session (including difficulty rating, suggested model
  size, and thinking budget). The agent proposes it, the human triggers
  it — it never runs unprompted. Do this *before* any compact: the delta
  check needs the uncompacted session.
- **Continue.** New chat, paste the block. After handoff the old chat is
  leftover — compacting it adds nothing. Claude Code may still `/compact`
  and stay (that harness re-injects `PROJECT.md` on compact); Cursor has
  no such hook. If Cursor auto-summarizes *mid-task*, the agent
  re-reads the index from disk. That is recovery, not session end.

## What lives where

Every file has a defined load behavior: always, targeted, or on demand.
Whatever loads always stays small — index files hold references, not
explanations.

> The file names below use the Claude Code harness as the running example
> (`CLAUDE.md`, `~/.claude/`, per-module `CLAUDE.md`). The tool-neutral
> contract is `AGENTS.md`; each harness maps the rest — with Cursor the
> equivalents are `AGENTS.md`, `~/.cursor/rules/` and `.cursor/rules/<area>.mdc`
> with globs. See `harnesses/` in the methodology repo.

| File | Read | Content |
|---|---|---|
| `CLAUDE.md` | every session, fully | Behavior rules only. No project knowledge. |
| `CLAUDE.local.md` | every session, fully | Personal rules in shared repos. Gitignored. |
| `docs/agentcontext/PROJECT.md` | every session, fully | Goal, status, to-dos, open decisions. Short, with references. |
| `docs/agentcontext/DETAILS.md` | targeted, single sections | One section per concept, with reasoning. May grow long. |
| `docs/agentcontext/plans/` | when working on that block | Approved plans for larger work blocks. Moved to `plans/done/` at block close. |
| `docs/agentcontext/HISTORY.md` | only when writing | Date, what, why. Optional. |
| `docs/agentcontext/TESTING.md` | on changes | What must work, as a checkable list. Optional. |

The split in one example — the reference lives in the index, the reasoning
next to it:

```
PROJECT.md
- [x] Grain decided (2026-08-12): one row = one request.
      Reason: polling not separable otherwise. → DETAILS "Modeling"

DETAILS.md
## Modeling
*(as of 2026-08-12)*
Three variants were considered … B was rejected because …
```

Reference-instead-of-explanation is the whole trick: PROJECT.md stays small
enough to always ride along, without losing knowledge.

## Instruction layers

Rules live on three layers, each with a defined audience:

- **`~/.claude/CLAUDE.md`** — the person: communication and working style,
  valid on this machine in every repo (maintained in this repo under
  `global/`).
- **`CLAUDE.md`** — the repo: committed. In team repos it is a team
  contract — it enforces only what the team agreed on, plus project facts.
- **`CLAUDE.local.md`** — the person in this repo: personal collaboration
  rules that shouldn't bind teammates. Gitignored, read automatically by
  Claude Code.

In solo projects the split doesn't matter — everything may live in
CLAUDE.md. The moment a repo is shared, personal rules move to
CLAUDE.local.md.

## Staying in control

- **Nothing bigger without an OK.** Plan first, the "yes" turns it into code.
- **No silent decisions.** Two viable paths → both are named, with a
  recommendation and reasoning. The human decides.
- **No hidden assumptions.** What the agent doesn't know, it asks — or writes
  down visibly as `> **ASSUMPTION:**` so it can be corrected.
- **Everything is in git.** The setup itself is a single commit; one command
  reverts the whole methodology.

## Principles

The rules the file structure follows from. They hold even if the concrete
files are named differently.

- **Behavior in CLAUDE.md, knowledge in the context files.** CLAUDE.md says
  how to work, never what the project is — no stack, no architecture, no
  status. Knowledge lives in PROJECT/DETAILS. Mixing the two makes both
  unreliable: rules drown in facts, facts go stale in a file nobody updates.
- **Every file has a lifespan.** Decide it per file, not per sentence: a file
  holds either permanent content (decisions with their reasons),
  current-until-superseded content (state and knowledge — updates *replace*,
  they never append), or block-scoped content (an active plan and its
  progress, which stops being authoritative when the block closes). Nobody maintains a "valid until" tag on every line. A
  lifespan per file makes misplaced content visible instead — a debugging story
  in a current-truth file is wrong by location, which you can see, rather than
  by judgment, which you have to make sentence by sentence.
- **Context is the scarce resource, not compute.** Defined load behavior per
  file; what always loads stays small.
- **The goal is a seamless restart.** The measure for every documentation
  decision: could a fresh agent with a small context continue from here? If
  not, something is missing in the files.
- **Persist immediately.** What should be documented at session end is already
  imprecise at session end. Write inline, don't batch.
- **Decisions stay with the human, work with the agent.** Control doesn't mean
  being asked more — it means every decision is visible, reversible, and none
  was taken silently.
- **The why is worth more than the what.** Summaries keep results and lose
  reasons. So reasons are written explicitly — for decisions, rules, and
  rejected alternatives.
- **Show options, then decide.** No path is presented as the only one.
  Separate "this is how you'd build it under conditions X" from "this is what
  we do for now, pragmatically".
- **Delegate heavy exploration.** Research that would pull many files into the
  main window runs in a subagent; only the condensed result comes back.
- **Extract skills from repetition.** When a procedure repeats across
  sessions, it becomes a skill — never speculatively, always after it proved
  itself. Keep the most important instructions at the top of a `SKILL.md`:
  after a compaction, invoked skill bodies are re-injected but truncated, and
  truncation keeps the beginning.
- **An archive is a graveyard, not a second knowledge base.** Finished plans
  are kept — an unread file costs nothing — but nothing gets updated there and
  nothing references it. The condition for archiving is that the learnings are
  already in DETAILS: otherwise the archive becomes an excuse to skip that
  step, and then there are two competing places for knowledge.
- **No document creep.** New files in `docs/agentcontext/` only when the content truly
  fits nowhere — then marked `> **TEMPORARY.**` with the condition for
  deleting it.
- **Deletion needs a moment, not good intentions.** Outdated assumptions are
  removed, not annotated — documentation you can't trust is worse than none.
  But generation is additive: an agent that writes on every turn grows every
  file, and an ambient "keep it short" loses to that bias every time. So
  removal is bound to an event — the close of a work block — and checked there
  like any other step.
- **Enforcement belongs where the bias is.** A rule that only sits in
  CLAUDE.md competes with the model's own tendencies. The few rules that
  reliably lose that fight — deleting, keeping the grain small — are anchored
  in a skill that runs at a defined moment. Deliberately not every rule: a
  ritual that grows into a forty-point audit gets skipped or rubber-stamped.
- **Global only after it proved itself.** New rules land in the project
  CLAUDE.md. Only what carried across several projects moves to
  `~/.claude/CLAUDE.md`.

## Scaling note

PROJECT + DETAILS with jump targets is deliberately centralized structured
note-taking. It scales until targeted jumping into DETAILS.md becomes
unreliable. Two growth paths, and they complement each other:

- **Knowledge scales via a `details/` folder.** When a DETAILS section
  outgrows targeted reading, extract it to
  `docs/agentcontext/details/<concept>.md` and leave only the reference.
  PROJECT.md stays the single index either way — its references just point to
  files instead of sections. DETAILS.md may eventually dissolve entirely into
  the folder.
- **Behavior scales via per-module `CLAUDE.md` files.** For module-specific
  rules (backend/, frontend/, …), place a small `CLAUDE.md` in that directory —
  Claude Code loads it automatically when files there are touched. Keep these
  tiny: local behavior rules plus pointers into `details/`, never knowledge
  dumps — otherwise there are two competing places for knowledge.

- **Rituals scale by decoupling from the session.** While one chat does one
  work block, the block ends when the session ends and `/handoff` is the only
  cleanup moment needed. Once several chats work one repo in parallel, blocks
  close mid-session and the global index has no owner — then block close
  becomes its own checklist, and one chat is explicitly responsible for the
  index.

The reference principle stays identical in all of them — only the loading
becomes mechanical instead of disciplined.

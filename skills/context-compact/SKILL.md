---
name: context-compact
description: Recovery after a mid-task compaction — Cursor auto-summarize or /summarize, Claude Code auto-compact or /compact — when the re-injected context is not enough. Re-reads project files from disk and optionally writes a short 5-point continue block. Not a session end — do not run this as part of /handoff. Use when the chat was just compacted, summarized, or the user says context was lost.
disable-model-invocation: true
argument-hint: "optional: what to continue"
---

# Context compact (recovery)

`/summarize` (Cursor) and `/compact` (Claude Code) are harness commands. A
skill cannot invoke them. This skill does not replace them and does not end a
session.

Harness note: Cursor has no hook that re-injects files after a summarize, so
this skill is the recovery path there. Claude Code re-loads `CLAUDE.md` (and
the `@AGENTS.md` it imports) natively and, if the SessionStart hook from
`harnesses/claude.md` is wired, re-injects `PROJECT.md` — run this skill there
only when that hook is missing or the agent still lacks what the next step
needs.

**When to run:** the chat was just compacted (manual or automatic) and
the agent must keep working *in this chat*. After a planned `/handoff`,
do not run this — the user starts a fresh chat (Claude Code: `/clear`)
with the handoff block.

**When not to run:** session end, "hand over", finished task. That is
`/handoff`.

A compact drops rules and files that were read earlier but never lived
in the user/assistant transcript. Re-inject from disk. Do not trust the
summary as the project index.

## Step 1 — Re-inject

Read, in this order, and only what exists:

1. This repo's `AGENTS.md` (session-start section is enough if the file
   is long)
2. `docs/agentcontext/PROJECT.md` — or, in the methodology repo itself,
   `agentcontext/PROJECT.md` if that is where the index lives
3. Git status: branch, last commit, uncommitted files. One short report
4. The last explicit user instruction (quote it if it is still in the
   preserved tail)
5. Skills needed for the *next* action — re-read their `SKILL.md` from disk;
   a compact can drop or truncate skill bodies loaded earlier

Do not dump DETAILS.md. Jump only into a section the next step needs.

## Step 2 — Continue block (only if there was no handoff)

If this chat already has a handoff block from `/handoff`, do not write a
second summary. Confirm re-inject and name the next step in one line.

Otherwise output this schema, short:

```markdown
## 1. Goal / user intent
[What this session is for]

## 2. Done
- [path]: [what changed or was decided]

## 3. Open / blockers
- …

## 4. Next step
> Quote the last user instruction if you have it

## 5. Decisions and constraints
- …
```

No tool calls while writing the block. No secrets.

## Step 3 — Close

One sentence: context restored from disk, next step is …. Then continue
the task. Do not propose `/handoff` unless the task is actually finished
or the window is still unusable after re-inject.

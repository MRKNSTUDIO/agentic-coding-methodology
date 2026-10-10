# Harness: Claude Code

Wire this when the user chooses `claude` (or `both`) in `/setup-project`. The
source of truth stays `AGENTS.md`; Claude Code gets a pointer and its own
skill/hook paths.

## Extra files

1. `CLAUDE.md` — short pointer, not a second contract. The `@AGENTS.md`
   line is a native Claude Code import: the contract is loaded every session
   and re-loaded after a compact, without the agent having to remember to
   read it.

   ```markdown
   # Claude Code — this repo uses AGENTS.md

   @AGENTS.md

   The import above loads `AGENTS.md` natively, every session and again after
   a compact. Do not duplicate rules here.

   Session start: `docs/agentcontext/PROJECT.md`.
   ```

   If `CLAUDE.md` already exists with real rules: merge into AGENTS.md,
   then replace CLAUDE.md with the pointer. Ask first.

2. `.claude/skills/handoff/SKILL.md` — same body as the Cursor copy.

3. SessionStart hook in `.claude/settings.json` (merge, never overwrite).
   The hook re-injects the project index after a compaction — the only hook
   event whose stdout reaches the model:

   ```json
   {
     "hooks": {
       "SessionStart": [
         {
           "matcher": "compact",
           "hooks": [
             {
               "type": "command",
               "command": "f=\"${CLAUDE_PROJECT_DIR}/docs/agentcontext/PROJECT.md\"; [ -f \"$f\" ] && { echo \"Context was just compacted. Project index, re-read from disk:\"; cat \"$f\"; }"
             }
           ]
         }
       ]
     }
   }
   ```

4. Gitignore `CLAUDE.local.md` and `.claude/settings.local.json`. Personal
   Claude rules and personal permissions go there in shared repos;
   `.claude/settings.json` (the hook) and `.claude/skills/` are committed.

Do **not** create `.cursor/` files unless the user also chose `cursor`.

## Compact

Happy path, same as Cursor: `/handoff`, then `/clear` (or a new session) and
paste the block as the first message. After the handoff the old transcript is
leftover.

Claude Code may also `/compact` and stay — mid-task, or when auto-compact
fires. That is safe here because `CLAUDE.md` (and with it `AGENTS.md`) is
re-loaded natively and the SessionStart hook re-injects `PROJECT.md`. Propose
`/handoff` *before* a planned compact anyway: the delta check needs the
uncompacted session. `/context-compact` is only needed if the hook is
missing or the re-injected index was not enough.

## User-level (once per machine)

```
<methodology-root>/skills/setup-project    →  ~/.claude/skills/setup-project
<methodology-root>/skills/update-project   →  ~/.claude/skills/update-project
<methodology-root>/skills/context-compact  →  ~/.claude/skills/context-compact   (optional)
<methodology-root>/skills/sync-upstream    →  ~/.claude/skills/sync-upstream     (maintainer only)
```

Link, don't copy: `setup-project` and `update-project` find the methodology
root by resolving their own symlink. Claude Code prints the skill's base
directory when the skill loads; resolve it with `realpath` (or
`readlink -f`) to reach the clone.

`~/.claude/CLAUDE.md` may `@`-import `global/AGENTS.md` and keep your name
and machine-private facts below the import.

## Module rules

Optional `backend/CLAUDE.md` (tiny: local behavior + pointers). Same
content can live as a Cursor rule with globs if both harnesses are on —
do not write the knowledge twice; pick one file and point the other at it.

---
name: sync-upstream
description: Pull Henroxx's methodology updates into this fork. Resolves and verifies the upstream remote, copies CLAUDE/ verbatim, and proposes translations onto the fork-owned files without losing declined deltas. Use in the methodology repo itself — scaffolded consumer repos use /update-project.
disable-model-invocation: true
argument-hint: "optional: what to focus the sync on"
---

# Sync Henroxx into this fork

This skill runs **only in the methodology repo** (this clone). Consumer
repos use `/update-project` against *this* clone's HEAD, not against
Henroxx.

If the current working tree is not the methodology repo (no
`templates/AGENTS.project.md` and no `harnesses/upstream.md`): stop. Tell
the user to run `/update-project` in that repo instead.

Read `harnesses/upstream.md` first — remotes, path map, kept-local list,
and the `vendor:` / `translated:` stamps.

Goal: refresh the vendor copy and propose translations — **without ever
overwriting local adaptations**. Local always wins; the sync proposes,
the user decides. Never merge upstream into this branch. Never push.

## Steps

1. **Read both stamps.**

   - `vendor: <hash> (<date>)` — what `CLAUDE/` mirrors
   - `translated: <hash> (<date>)` — last fully resolved translation

   Missing `vendor:` means the folder has never been copied. Missing
   `translated:` means translate from the oldest available upstream commit
   and say that the baseline had to be inferred.

2. **Resolve and verify the upstream remote.** Canonical repository identity:
   `Henroxx/agentic-coding-methodology`.

   - If `AGENTIC_UPSTREAM_REMOTE` names a remote, verify its fetch URL first.
   - Otherwise inspect fetch URLs from `git remote -v` and select the unique
     remote whose normalized HTTPS or SSH URL identifies the canonical repo
     (ignore protocol, optional `.git`, case, and trailing slash).
   - No match: stop and show the `git remote add upstream ...` command from
     `harnesses/upstream.md`.
   - More than one match: stop and ask which remote to use.

   Call it `<upstream-remote>` below. Never assume it is named `origin`.

3. **Protect the vendor tree.** Before fetching or replacing anything:

   ```
   git status --porcelain -- CLAUDE
   ```

   Any output means staged or unstaged vendor changes exist. Stop and report
   them; never overwrite, stash, restore, or discard them automatically.

4. **Fetch and identify the target.**

   ```
   git fetch <upstream-remote>
   git rev-parse --short <upstream-remote>/main
   ```

   Also read that commit's date. If both stamps equal the target and
   `CLAUDE/` matches `<upstream-remote>/main`, report "up to date" and stop.

5. **Collect every unresolved delta.** Diff from `translated:`, not
   `vendor:`. Include root `CLAUDE.md` for comparison, but never vendor it:

   ```
   git diff <translated>..<upstream-remote>/main -- CLAUDE.md CLAUDE/templates CLAUDE/skills CLAUDE/variants CLAUDE/METHODOLOGY.md CLAUDE/global
   ```

   This deliberately repeats previously accepted items while any temporary
   decline remains. Compare with the fork-owned target files and omit
   translations that are already present.

6. **Replace `CLAUDE/` verbatim.** Restore his folder only:

   ```
   git restore --source=<upstream-remote>/main --staged --worktree -- CLAUDE
   git diff --exit-code <upstream-remote>/main -- CLAUDE
   ```

   The verification must exit cleanly. Otherwise stop; do not update either
   stamp. On success, update `vendor:` to the target hash and date. Do not
   checkout his root `README.md` or root `CLAUDE.md`.

7. **Translate.** Read the diff and the path map in
   `harnesses/upstream.md`. Turn each applicable change into a concrete
   proposed patch on **fork-owned files**. Ignore or route to
   `harnesses/claude.md`:

   - SessionStart compact hook
   - `CLAUDE.md` as the contract (he has no `AGENTS.md`)
   - `~/.claude/` paths as the only wiring

   Items on the kept-local list stay local — list them as "kept local",
   never silently revert. That includes the Cursor continue loop
   (handoff → fresh chat, not compact-and-stay).

8. **Propose, don't apply.** Numbered list — per item: which fork-owned
   file, what changes, why. Close with: "Which numbers should I apply?"

9. **Apply and restamp safely.** Apply only the chosen items.

   - Every applicable delta through the target is now applied or recorded in
     `harnesses/upstream.md` as deliberately kept local: advance
     `translated:` to the target hash and date.
   - Any item is declined only for now: leave `translated:` unchanged, so it
     appears next time. Never hide pending work in chat-only state.

   Do not change consumer `methodology:` stamps. Do not commit or push unless
   the user explicitly asks. If committing, stage exact paths only — never
   `git add -A`, `git add .`, or `git commit -a`.

## Rules

- Never edit files under `CLAUDE/` except by the verified restore above.
- Never touch filled slots or project knowledge in consumer
  `docs/agentcontext/` — this skill does not run there.
- Never advance `vendor:` without a verified exact copy.
- Never advance `translated:` while a temporary decline remains.
- Never modify git remotes automatically.

---
name: update-project
description: Checks whether the current repo's methodology scaffold is up to date with the methodology repo, and proposes the deltas. Use when the user asks whether the setup/methodology is current, or wants to pull methodology updates into this repo.
disable-model-invocation: true
argument-hint: "optional: what to focus the update on"
---

# Update the methodology in this repo

If this working tree *is* the methodology repo (`templates/AGENTS.project.md`
and `harnesses/upstream.md` exist): stop. This skill is for scaffolded
consumer repos only. For Henroxx / upstream updates here, use
`/sync-upstream`.

Resolve **methodology root** (first match):

1. Environment variable `AGENTIC_METHODOLOGY_ROOT`
2. The resolved target of this installed skill's symlink/junction: from
   `skills/update-project/SKILL.md`, go up three levels and verify that the
   result contains `templates/AGENTS.project.md`. Claude Code names the
   skill's base directory when it loads — resolve that path with `realpath`
   (or `readlink -f`); Cursor: resolve `~/.cursor/skills/update-project`
3. The current repo, if it contains `templates/AGENTS.project.md`

If the skill was copied rather than linked and none match, stop and ask the
user to set `AGENTIC_METHODOLOGY_ROOT` to their clone. Never guess a
machine-specific path.

Goal: bring this repo's scaffold (`AGENTS.md`, `docs/agentcontext/`,
harness skill copies) up to date with the methodology — **without ever
overwriting local adaptations**. Local always wins; the update proposes,
the user decides.

## Steps

1. **Read the stamp.** The last line of this repo's `AGENTS.md` is
   `methodology: <hash> (<date>)`. No stamp → look at a leftover `CLAUDE.md`
   only if `AGENTS.md` is missing; otherwise the repo predates versioning or
   was never scaffolded. Say so and offer `/setup-project` (it merges with
   existing files) instead of continuing here.

2. **Compare.** Get the current version:
   `git -C <methodology-root> rev-parse --short HEAD`.
   If it equals the stamp: report "up to date" and stop.

3. **Collect the deltas.** Diff what affects scaffolded repos:

   ```
   git -C <methodology-root> \
       diff <stamp>..HEAD -- templates skills variants harnesses
   ```

   Infer this repo's harness from disk (see `harnesses/README.md`):
   `.cursor/skills/` → cursor; `CLAUDE.md` pointer or `.claude/skills/` →
   claude; both → both. Do not re-ask. For claude, also compare the
   `CLAUDE.md` pointer and the SessionStart hook in `.claude/settings.json`
   against `harnesses/claude.md` — the hook only changes via a merge, never
   by overwriting other settings.

   Read the diff and translate it into concrete proposed changes to *this*
   repo's files. Ignore changes that don't apply here (e.g. a variant or
   harness this repo doesn't use, template sections the setup removed on
   purpose).

4. **Propose, don't apply.** Present the deltas as a numbered list — per
   item: which file, what changes, why (from the methodology's reasoning).
   Where a local deviation collides with an upstream change, say so
   explicitly and default to **keeping the local version** — deviations
   that look deliberate are listed as "kept local", never silently
   reverted. Close with: "Which numbers should I apply?"

5. **Apply and restamp.** Apply the chosen items. Update the stamp line to
   the new hash and date. One commit for the whole update run, respecting
   this repo's git permissions in AGENTS.md.

## Rules

- Never touch project content: filled slots, project knowledge in
  docs/agentcontext/, project-specific rules. Only structural and rule
  deltas that came from the methodology.
- If `.cursor/skills/handoff/SKILL.md` (or a Claude copy under
  `.claude/skills/handoff/`) was customized locally, show the diff against
  the new template instead of replacing it.
- No stamp update without applied changes — if the user declines
  everything, the stamp stays, so the deltas show up again next time.

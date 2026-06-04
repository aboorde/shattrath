# Example prompt — audit & refactor a repo's setup

*A copy-paste prompt that points Claude Code at this kit and drives an audit → plan → refactor of a repo's `CLAUDE.md` / `.claude/rules/` setup. Compiled 2026-06-04; tracks the re-verified kit.*

Use this when you pull this repo down and want to turn another repo's neglected, `/init`-once `CLAUDE.md` (plus existing internal plugin skills) into an effective, scalable instruction architecture.

## How to use

1. **Replace `<CLAUDE_RULES_DIR>`** with the real path to this folder (e.g. `~/code/shattrath/claude_rules`).
2. **Give Claude access to the docs**, since they live in a different repo than your work repo. Start Claude *in the work repo* and add the docs directory: `claude --add-dir ~/code/shattrath/claude_rules` (or just use the absolute path in the prompt — Claude reads the files either way).
3. **Run in plan mode** so the audit + plan land before any edits: press `Shift+Tab` to cycle into Plan mode, or start with `claude --permission-mode plan`. The prompt also tells Claude to stop after Phase 2 for approval.

> Want this as a reusable `/claude-rules` command? Drop the prompt body into a skill (`~/.claude/skills/claude-rules/SKILL.md`) with the docs path baked in, and invoke it by name.

## The prompt

```text
You are refactoring this repo's Claude Code instruction setup into an effective, scalable
architecture. Treat the docs in <CLAUDE_RULES_DIR> as the authoritative best-practice spec.

INPUTS — read in this order before doing anything:
1. <CLAUDE_RULES_DIR>/audit-rubric.md  (this is your grading checklist — use its dimension/ID/severity scheme)
2. <CLAUDE_RULES_DIR>/choosing-the-right-mechanism.md  (the routing rubric for where each instruction belongs)
3. <CLAUDE_RULES_DIR>/claude-md-anatomy.md, rules-directory.md, adherence-and-anti-patterns.md,
   context-and-token-economics.md, large-codebase-patterns.md  (skim; cite IDs/principles you apply)
Then inventory THIS repo's current setup:
- The existing CLAUDE.md (and any nested CLAUDE.md, .claude/rules/, @imports, CLAUDE.local.md).
- Our internal plugins/skills: enumerate every skill (name + description + what it enforces). These ALREADY
  hold our code standards/practices — do NOT copy their content into CLAUDE.md; treat them as the on-demand layer.
- Enough of the actual codebase to ground recommendations (languages, package layout / monorepo?, build &
  test commands, where conventions live, generated/vendored dirs).

DO — PHASE 1, AUDIT (no edits yet):
- Run the audit-rubric against the current setup. Report findings as [ID · SEVERITY] file:line — evidence — fix.
- Lead with Dimension A: flag any must-hold/safety/security rule living only as CLAUDE.md prose with no
  PreToolUse hook / permissions.deny backing.
- Specifically check the CLAUDE.md↔skills boundary: content in CLAUDE.md that duplicates or contradicts a
  plugin skill; conventions that should be a skill or a path-scoped rule, not always-on prose; skill
  descriptions that won't survive truncation (combined description + when_to_use is capped at 1,536 chars —
  lead with the keywords a real request would contain); /init bloat and stale model-workaround rules.
- Mark anything that needs a live session (whether files actually load, token cost, dead skills) as
  "requires runtime verification" — and ask me to paste `/memory`, `/context`, and `/status` output if useful.

DO — PHASE 2, PLAN:
Propose the target architecture and STOP for my approval before writing files. Show:
- A slimmed root CLAUDE.md (<200 lines: only non-inferable, behavior-shifting facts; positive, scoped, why-attached).
- Which content moves to .claude/rules/ (which path-scoped vs unconditional), to our existing skills, or to new
  hooks/permissions.deny (for anything that must hold every time); for the rare behavioral rule that needs
  system-prompt priority but isn't hook-enforceable, note --append-system-prompt or managed claudeMd.
- If the repo is large or typed and Claude greps to find where symbols live, recommend a code-intelligence
  (LSP) plugin in the plan (the routing ladder's newest official lever — it's setup/config, not app code).
- For a monorepo: two-level layering (root + per-package) and starting-directory guidance.
- A diff-style before/after of CLAUDE.md and a file list of what gets created/moved/deleted.

DO — PHASE 3, IMPLEMENT (only after I approve the plan):
Make the edits. Keep our plugins/skills as the source of truth for code standards — CLAUDE.md should reference
and route to them, not restate them. Re-run the rubric to confirm findings are resolved.

CONSTRAINTS:
- CLAUDE.md/rules are guidance, not enforcement — route guarantees to hooks/permissions, don't trust prose.
- Don't invent build/test/lint tooling or commands; discover the real ones from the repo.
- Cite the audit-rubric IDs and best-practice principles behind each change.
- Don't touch application code; only the Claude Code instruction layer (CLAUDE.md, .claude/rules/, hooks/settings,
  and skill descriptions if they're under-discoverable).
```

Replace `<CLAUDE_RULES_DIR>` with the actual path (e.g. `~/code/shattrath/claude_rules`).

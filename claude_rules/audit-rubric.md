# CLAUDE.md & .claude/rules/ Audit Rubric

*An operational scoring checklist for an Opus agent to grade a repository's Claude Code instruction setup against best practice. Compiled 2026-06-04. Each check states how to detect it, a severity, the best-practice basis (sourced), and a remediation. Pairs with [choosing-the-right-mechanism.md](choosing-the-right-mechanism.md) (the routing rubric the placement checks enforce).*

---

## How to use this doc

You are auditing a repo's `CLAUDE.md`, `.claude/rules/`, nested memory files, `@imports`, and adjacent enforcement config. Produce a graded report, not prose praise. Two classes of check, **which must not be conflated** (source: https://code.claude.com/docs/en/debug-your-config):

- **Static** — checkable by reading the repo alone (files, line counts, frontmatter, globs, model IDs, secrets, contradictions, placement). This is the bulk of the rubric and what community linters encode (source: https://github.com/agent-sh/agnix; source: https://github.com/felixgeelhaar/cclint).
- **Runtime** — only knowable from a live session or telemetry: *did a file actually load* (`/memory`, `InstructionsLoaded` hook), *what does it cost* (`/context`), *is managed policy in effect* (`/status`), *is a skill/rule dead* (OpenTelemetry). If you have no live session, report these as **"requires runtime verification"**, do not score them as pass (source: https://code.claude.com/docs/en/debug-your-config; source: https://code.claude.com/docs/en/monitoring-usage).

### Audit procedure
1. **Inventory.** Enumerate every instruction file Claude *would* load: project `./CLAUDE.md` / `./.claude/CLAUDE.md`, every ancestor and nested `CLAUDE.md`/`CLAUDE.local.md`, all `.claude/rules/**/*.md` (recursive), every `@import` target (follow ≤4 hops), `~/.claude/CLAUDE.md` and `~/.claude/rules/` if in scope, and any managed-policy file. Note which are committed vs gitignored.
2. **Budget.** Estimate total always-loaded instruction tokens (CLAUDE.md + unconditional rules + all `@import` expansions — imports are inline, not lazy). See [context-and-token-economics.md](context-and-token-economics.md).
3. **Run the checks below**, recording each finding as `{id, severity, file:line, evidence, fix}`.
4. **Score & rank.** Emit the scorecard and the top 3 highest-severity fixes.

---

## Severity scale

| Severity | Meaning | Examples |
|---|---|---|
| **BLOCKER** | A safety/security guarantee is written as advisory prose, or a secret is committed. Will cause incidents. | "Never push to main" only in CLAUDE.md; committed API key |
| **HIGH** | Materially degrades adherence or correctness for everyone. | 1,100-line root CLAUDE.md; contradictory rules; AGENTS.md with no CLAUDE.md bridge |
| **MEDIUM** | Wastes context or invites drift; fix improves reliability. | `@import` used "to save tokens"; unscoped rule that should be path-scoped; stale model-workaround |
| **LOW** | Hygiene/polish; small token or maintainability win. | Generic rule filenames; HTML-comment notes missing; emphasis on a non-critical rule |
| **INFO** | Requires runtime/telemetry to confirm; report, don't fail. | Whether a path-scoped rule loads; dead-skill detection |

---

## Dimension A — Placement & enforcement (weight: highest)

The single most important audit axis. CLAUDE.md and rules are **context, not enforced configuration**; an instruction is "a request, not a guarantee" — only hooks and `permissions.deny` are deterministic (source: https://code.claude.com/docs/en/memory; source: https://code.claude.com/docs/en/features-overview).

| ID | Check | Detect | Severity | Fix |
|---|---|---|---|---|
| **ENF-01** | A safety/destructive/security rule (`never push to main`, `never edit .env`, `never delete prod data`, `always run tests before commit`) lives **only** in CLAUDE.md/rules prose, with no backing `PreToolUse` hook or `permissions.deny`. | Static: grep for imperative-guarantee phrasing (never/always/must) on destructive or security topics; cross-check `.claude/settings*.json` `permissions.deny` and `.claude/hooks` / `hooks` settings for a matching guard. | **BLOCKER** | Move the guarantee to a `PreToolUse` hook (`exit 2` / `permissionDecision:"deny"`) or `permissions.deny`; keep the prose as rationale only (source: https://code.claude.com/docs/en/hooks-guide; issue documenting the failure: https://github.com/anthropics/claude-code/issues/5502). |
| **ENF-02** | "Must run at a point" rule (lint/format/test on every edit/commit) written as prose instead of a hook. | Static: prose like "always run prettier after editing" with no `PostToolUse` hook. | HIGH | Encode as a `PostToolUse`/`Stop` hook — "Unlike CLAUDE.md instructions which are advisory, hooks are deterministic" (source: https://code.claude.com/docs/en/best-practices). |
| **ENF-03** | Org-wide non-negotiables (security policy, login org lock) deployed via project CLAUDE.md rather than **managed** settings. | Static (if managed config visible): security rules in `./.claude/` not in managed policy. | HIGH | Put hard enforcement in managed settings (`permissions.deny`, `sandbox.enabled`, `forceLoginMethod`) and behavioral guidance in managed CLAUDE.md (source: https://code.claude.com/docs/en/memory; source: https://code.claude.com/docs/en/admin-setup). |
| **ENF-04** | A `PostToolUse` (or other post-event) hook is used to "block" an action it cannot undo. | Static: post-event hook returning deny intent. | MEDIUM | Only `PreToolUse`/`UserPromptSubmit` can block; move the guard earlier (source: https://code.claude.com/docs/en/hooks-guide). |

> **ENF-01 is the top-tier finding.** A repo can score well on every other dimension and still be dangerous if its guardrails are prose. Always check it first.

---

## Dimension B — Size & token budget

CLAUDE.md is loaded **in full every session**; "longer files consume more context and reduce adherence" (source: https://code.claude.com/docs/en/memory). See [context-and-token-economics.md](context-and-token-economics.md) for measurement.

| ID | Check | Detect | Severity | Fix |
|---|---|---|---|---|
| **SIZE-01** | Any single `CLAUDE.md` exceeds the **200-line** target. | Static: `wc -l`. 200–400 = MEDIUM; >500 = HIGH; >1,000 = HIGH/canonical-bad. | MEDIUM→HIGH | Apply the prune test; move file-type content to path-scoped `.claude/rules/`, procedures to skills (source: https://code.claude.com/docs/en/memory; community thresholds: https://www.digitalapplied.com/blog/claude-code-team-adoption-audit-50-point-scorecard-2026). |
| **SIZE-02** | `@import` used believing it "saves context." Imports expand inline at launch; count them in the budget. | Static: `@path` lines; sum imported file sizes into the CLAUDE.md budget. | MEDIUM | Use a path-scoped rule or skill for deferral; reserve `@import` for always-needed shared content (source: https://code.claude.com/docs/en/memory). |
| **SIZE-03** | `@import` chain deeper than **4 hops** (silently breaks). | Static: follow imports, count depth. | MEDIUM | Flatten to ≤4 hops (source: https://code.claude.com/docs/en/memory). |
| **SIZE-04** | Embedded code snippets / pasted API docs / full file-by-file descriptions in CLAUDE.md. | Static: fenced code blocks and long reference dumps. | MEDIUM | Replace with `file:line` references and links; they go stale and inflate the always-loaded budget (community: https://www.humanlayer.dev/blog/writing-a-good-claude-md). |
| **SIZE-05** | `MEMORY.md` (auto memory index) exceeds **200 lines / 25 KB** — content past the cap silently never loads. | Static (if present): `wc -l`/byte size of `~/.claude/projects/<project>/memory/MEMORY.md`. | MEDIUM | Keep MEMORY.md a thin index; push detail to topic files (source: https://code.claude.com/docs/en/memory). See [auto-memory-and-self-improvement.md](auto-memory-and-self-improvement.md). |
| **SIZE-06** | Total always-loaded instruction budget is large relative to the window. | Runtime: `/context` per-category breakdown. | INFO | Trim the dominant category (often built-in tools/file-reads, not CLAUDE.md) (community capture: https://wmedia.es/en/tips/claude-code-context-command-token-usage). |

---

## Dimension C — Structure & modularity

| ID | Check | Detect | Severity | Fix |
|---|---|---|---|---|
| **STRUCT-01** | Monorepo with one monolithic root CLAUDE.md covering every subsystem (loads everywhere; frontend sessions carry backend rules). | Static: large root + multiple packages with no per-package CLAUDE.md. | HIGH | Two-level layering: root = repo-wide only (<200 lines), per-package CLAUDE.md owned by directory owners (source: https://code.claude.com/docs/en/large-codebases). See [large-codebase-patterns.md](large-codebase-patterns.md). |
| **STRUCT-02** | Package-specific rules have **leaked up** into the root CLAUDE.md. | Static: root contains stack/command details that only apply to one package. | MEDIUM | Move them down into the owning package's CLAUDE.md. |
| **STRUCT-03** | Heavy file-type-specific guidance sits in an **unconditional** rule/CLAUDE.md instead of a path-scoped rule. | Static: rule files with no `paths:` frontmatter whose content is clearly area-specific. | MEDIUM | Add `paths:` frontmatter so it loads only on matching files — but see RULE-GOTCHA below. |
| **STRUCT-04** | `.claude/rules/` files use generic names (`rules1.md`, `stuff.md`) or a flat pile. | Static: filename inspection. | LOW | One topic per file, self-describing names; subdirs by layer/domain (community: https://claude-blog.setec.rs/blog/claude-code-rules-directory). |
| **STRUCT-05** | Personal preferences committed in shared `CLAUDE.md` instead of `CLAUDE.local.md`/user scope. | Static: first-person/sandbox-URL/test-data lines in committed file. | LOW | Move to `CLAUDE.local.md` (gitignored) or `~/.claude/CLAUDE.md` (source: https://code.claude.com/docs/en/memory). |

### Path-scoped-rule reliability (RULE-GOTCHA — version-fragile, INFO/runtime)
Path-scoped `.claude/rules/` triggering has shipped broken **both ways** and is blind to file creation. Do not treat `paths:` as a guarantee; verify with `/memory` or `/context` on the running version.

| ID | Check | Detect | Severity | Fix |
|---|---|---|---|---|
| **RULE-01** | Convention that must apply to **newly created** files relies on a path-scoped rule. | Static: `paths:` rule whose intent is "all new files under X must …". | HIGH | Path rules trigger on the pre-edit **Read**, so they fire on edit but **not on `Write` of a new file** — "blind to file creation" (open issue, closed not-planned: https://github.com/anthropics/claude-code/issues/38487). Keep creation-critical conventions unscoped or enforce with a `PreToolUse` hook on `Write`. |
| **RULE-02** | Setup assumes path-scoped rules reliably load. | Runtime: `/memory`/`/context` after launch. | INFO | Two open contradictory bugs — never loads in subdirs (https://github.com/anthropics/claude-code/issues/16853, v2.1.1) and loads globally regardless of `paths:` (https://github.com/anthropics/claude-code/issues/16299, v2.0.76). Verify on the actual version; keep critical guidance unscoped until confirmed. |
| **RULE-03** | Path-scoped rules placed in `~/.claude/rules/` (user level). | Static. | MEDIUM | User-level path-scoped rules are reported silently ignored and are the lowest-priority layer; keep path-scoped rules at project level (https://github.com/anthropics/claude-code/issues/16853). |

---

## Dimension D — Content quality & adherence

Anchored on the official prune test and prompt-engineering guidance. See [adherence-and-anti-patterns.md](adherence-and-anti-patterns.md).

| ID | Check | Detect | Severity | Fix |
|---|---|---|---|---|
| **CONT-01** | Lines that fail the official test: *"Would removing this cause Claude to make mistakes? If not, cut it."* (inferable-from-code facts, standard conventions, "write clean code", file-tree narration). | Static: scan for self-evident advice, restated language defaults, file-by-file descriptions. | MEDIUM | Cut to non-inferable facts only (source: https://code.claude.com/docs/en/best-practices). |
| **CONT-02** | Negative phrasing (`don't`, `NEVER use X`) where a positive directive would work. | Static: count negative imperatives. | LOW→MEDIUM | Reframe to "do Y instead" + a positive example — official guidance prefers telling Claude what to do (source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices); the ironic-process "pink elephant" effect makes prohibitions leak (community: https://eval.16x.engineer/blog/the-pink-elephant-negative-instructions-llms-effectiveness-analysis). |
| **CONT-03** | Vague rules ("format properly", "test your changes") instead of verifiable ones ("use 2-space indentation", "run `npm test` before committing"). | Static. | MEDIUM | Make every rule concrete enough to verify (source: https://code.claude.com/docs/en/memory). |
| **CONT-04** | Emphasis inflation — `IMPORTANT`/`YOU MUST`/ALL-CAPS on many rules. | Static: count emphasized rules. | LOW | Emphasis is an official tuning lever but only as a **scarce** signal; reserve for 1–2 load-bearing rules (source: https://code.claude.com/docs/en/best-practices). |
| **CONT-05** | Rules that depend on the model generalizing (no explicit scope). | Static: rules like "use named exports" with no "everywhere/for every component". | LOW | State scope explicitly — Opus interprets literally and "does not silently generalize" (source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices). |
| **CONT-06** | LLM rule doing a linter's/formatter's job ("always 2-space indent", "no trailing whitespace"). | Static. | MEDIUM | Enforce with a `PostToolUse`/pre-commit hook and delete the rule — "never send an LLM to do a linter's job" (community: https://www.humanlayer.dev/blog/writing-a-good-claude-md). |
| **CONT-07** | Raw `/init` output committed verbatim (kitchen-sink bloat). | Static: file-tree narration, generic advice, no curation. | MEDIUM | Treat `/init` as a draft; prune against the include/exclude table. Auto-generated context files measured **net-negative** (reduce success, +20% cost) (source: https://arxiv.org/abs/2602.11988). |

---

## Dimension E — Consistency & freshness

| ID | Check | Detect | Severity | Fix |
|---|---|---|---|---|
| **FRESH-01** | Contradictory instructions across root / nested / rules / user files (Claude picks one arbitrarily). | Static: diff rules across all loaded files for conflicts (indentation, tooling, workflow). | HIGH | Reconcile or delete; periodic review is official guidance (source: https://code.claude.com/docs/en/memory). |
| **FRESH-02** | Stale **model-workaround** rules (e.g. "force single-file refactors", "split large edits") that newer models no longer need. | Static: rules describing model limitations; check repo/model era. | MEDIUM | Revisit after major model releases — such rules "become overhead" (source: https://code.claude.com/docs/en/best-practices; blog cadence 3–6 months: https://claude.com/blog/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start). |
| **FRESH-03** | Deprecated model IDs hard-coded (e.g. `claude-3-5-sonnet`) in CLAUDE.md / subagent / hook config. | Static: grep model IDs against current set. | MEDIUM | Update to current IDs; community linters flag this (https://github.com/felixgeelhaar/cclint). |
| **FRESH-04** | No sign of maintenance (last-edited long ago; not reviewed in PRs). | Static: git log of instruction files. | LOW | Treat CLAUDE.md edits as reviewed docs; >90 days stale is a community red flag (https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/tools/audit-prompt.md). |

---

## Dimension F — Interop & layout

| ID | Check | Detect | Severity | Fix |
|---|---|---|---|---|
| **INT-01** | Repo has `AGENTS.md` (or `.cursorrules`/`.windsurfrules`) but **no `CLAUDE.md` bridge** — Claude Code reads `CLAUDE.md`, not `AGENTS.md`. | Static: presence of `AGENTS.md` and absence of a `CLAUDE.md` importing it. | HIGH | Add `CLAUDE.md` with `@AGENTS.md` (or a symlink; on Windows prefer the `@import`) (source: https://code.claude.com/docs/en/memory). See [claude-md-anatomy.md](claude-md-anatomy.md). |
| **INT-02** | Duplicated, drifting instructions across `CLAUDE.md` and `AGENTS.md` instead of one source + bridge. | Static: overlapping content. | MEDIUM | Single source of truth + `@import` bridge. |
| **INT-03** | `CLAUDE.local.md` not gitignored. | Static: check `.gitignore`. | LOW | Add to `.gitignore` (source: https://code.claude.com/docs/en/memory). |

---

## Dimension G — Auto-memory hygiene (if present)

See [auto-memory-and-self-improvement.md](auto-memory-and-self-improvement.md). Auto memory is **machine-local** and **manually pruned only** — a wrong learning persists every session until a human deletes it (no shipped auto-prune; request closed not-planned: https://github.com/anthropics/claude-code/issues/37102).

| ID | Check | Detect | Severity | Fix |
|---|---|---|---|---|
| **MEM-01** | Stale/incorrect entries in `~/.claude/projects/<project>/memory/`. | Runtime/local: read topic files via `/memory`. | INFO→MEDIUM | Audit like a dependency; delete wrong entries (source: https://code.claude.com/docs/en/memory). |
| **MEM-02** | Team relies on auto memory to share knowledge (it never reaches teammates/CI). | Static/interview. | MEDIUM | Promote stabilized facts into version-controlled CLAUDE.md/rules; for agents use subagent `memory: project` (git-trackable) (source: https://code.claude.com/docs/en/memory; source: https://code.claude.com/docs/en/sub-agents). |
| **MEM-03** | Auto-consolidated ("dreamed") memory trusted without review. | Static/interview. | MEDIUM | `Dreams`/auto-dream is a Research Preview, partially/buggily wired in Claude Code as of 2026-06 (https://github.com/anthropics/claude-code/issues/38461; https://github.com/anthropics/claude-code/issues/38493) — require human review. |

---

## Dimension H — Security & secrets

| ID | Check | Detect | Severity | Fix |
|---|---|---|---|---|
| **SEC-01** | Secrets/PII/credentials committed in `CLAUDE.md`, rules, skills, or hooks. | Static: grep for `API_KEY|SECRET|PASSWORD|PRIVATE_KEY|TOKEN|Authorization: Bearer`, `.pem`, `.env` values. | **BLOCKER** | Remove and rotate; ensure `.env*`/`*.pem`/`credentials*` are in `permissions.deny` Read (source: https://code.claude.com/docs/en/security; https://github.com/anthropics/claude-code/issues/21674). |
| **SEC-02** | `~/.claude/CLAUDE.md` integrity — a writable, always-loaded prompt-injection vector any user process can append to. | Local: ownership, perms, modification time. | HIGH (env) | Check perms/ownership; treat unexpected edits as injection (open issue, closed not-planned: https://github.com/anthropics/claude-code/issues/21674). |
| **SEC-03** | `@import`ed or fetched content treated as trusted. Imported files can carry injected instructions. | Static: imports of files outside the repo's trust boundary. | MEDIUM | Treat imported/fetched content as untrusted; the first external import triggers an approval dialog by design (source: https://code.claude.com/docs/en/memory; source: https://code.claude.com/docs/en/security). |
| **SEC-04** | `permissions.deny WebFetch` used for egress control while Bash `curl`/`wget` is allowed. | Static: settings. | MEDIUM | Close with `sandbox.network.allowedDomains` (OS-level), not just tool denies (source: https://code.claude.com/docs/en/admin-setup). |
| **SEC-05** | Plaintext memory/transcript store at `~/.claude/projects/` may contain leaked secrets. | Local: grep the store. | INFO | World-readable, no retention policy; grep and tighten perms (community: https://serendb.com/blog/claude-code-local-memory-security-risk). |

---

## Dimension I — Governance & fleet (org rollout)

Mostly runtime/telemetry; report as INFO unless config is visible. See [large-codebase-patterns.md](large-codebase-patterns.md).

| ID | Check | Detect | Severity | Fix |
|---|---|---|---|---|
| **GOV-01** | Org has no managed-policy baseline; everything is per-repo and excludable. | Runtime: `/status` shows whether managed settings are in effect (and source tag). | INFO | Deploy managed-policy CLAUDE.md (non-excludable) + managed settings (source: https://code.claude.com/docs/en/admin-setup). |
| **GOV-02** | Customization not locked to plugins/managed where required. | Static (managed config): absence of `strictPluginOnlyCustomization`, `allowManagedPermissionRulesOnly`, `permissions.disableBypassPermissionsMode`. | INFO | Apply managed-only lockdown switches (source: https://code.claude.com/docs/en/settings). |
| **GOV-03** | Dead skills/rules/MCP accumulating (no usage signal). | Runtime/OTEL: distinct `skill.name`/`plugin.name`/`mcp_server.name` on events (needs `OTEL_LOG_TOOL_DETAILS=1`). | INFO | A skill present in-repo but never appearing across the fleet is dead — retire it (source: https://code.claude.com/docs/en/monitoring-usage). |
| **GOV-04** | No owner of the root / no review cadence. | Static/interview. | LOW | Assign a DRI/agent-manager; review config every 3–6 months (source: https://claude.com/blog/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start). |

---

## Runtime verification checklist (when a live session is available)

| Command / hook | Confirms |
|---|---|
| `/memory` | Which `CLAUDE.md`, `CLAUDE.local.md`, and rules files actually loaded; auto-memory entries (source: https://code.claude.com/docs/en/debug-your-config). |
| `/context` | Per-category token cost — is CLAUDE.md/rules the real overhead, or system tools/file reads? (source: https://code.claude.com/docs/en/context-window). |
| `/status` | Whether managed settings are in effect, and their source (source: https://code.claude.com/docs/en/admin-setup). |
| `/doctor` | Schema/validation errors, invalid keys, description-budget overflow (source: https://code.claude.com/docs/en/debug-your-config). |
| `InstructionsLoaded` hook | Programmatic log of every loaded file with `memory_type` + `load_reason` (`session_start`/`nested_traversal`/`path_glob_match`/`include`/`compact`) — catches "subdir CLAUDE.md silently not loading" that static reads miss (source: https://code.claude.com/docs/en/hooks). |
| Clean-config bisection | `CLAUDE_CONFIG_DIR=<empty>` + launch from a dir with no `.claude/`/`CLAUDE.md` isolates whether a behavior comes from config (source: https://code.claude.com/docs/en/debug-your-config). |

---

## Output format (what the auditing agent should emit)

```
## Scorecard
| Dimension | Findings | Worst severity |
|---|---|---|
| A Placement & enforcement | n | BLOCKER/… |
| … | | |

## Findings
- [ENF-01 · BLOCKER] CLAUDE.md:42 — "Never push to main" has no PreToolUse/permissions.deny backing.
  Evidence: <quote>. Fix: add deny rule / PreToolUse hook.
- [SIZE-01 · HIGH] CLAUDE.md — 1,140 lines (target <200). Fix: prune + path-scope.
- …

## Top 3 fixes (ranked by severity × blast radius)
1. …

## Requires runtime verification
- RULE-02, SIZE-06, GOV-03, MEM-01 — need /memory, /context, /status, or OTEL.
```

Rank fixes by **severity × blast radius** (a BLOCKER in the root file outranks a HIGH in one package). Always lead with Dimension A.

---

## Prior art (replicate or run these)

This rubric synthesizes and tightens existing community graders; an Opus auditor can also run them directly:

- **agnix** — linter/LSP/CI-Action for `CLAUDE.md`/`AGENTS.md`/`SKILL.md`/hooks/MCP (hundreds of rules; tiered autofix `--fix-safe`/`--fix`/`--fix-unsafe`) (source: https://github.com/agent-sh/agnix).
- **cclint** — concrete numeric thresholds and rule IDs (file-size, required sections, import depth/cycles, command-safety, deprecated model IDs) (source: https://github.com/felixgeelhaar/cclint; second implementation: https://github.com/carlrannaberg/cclint).
- **50-point Team Adoption Scorecard** — 5 axes, maturity tiers, grades against repo *evidence* not self-report (source: https://www.digitalapplied.com/blog/claude-code-team-adoption-audit-50-point-scorecard-2026).
- **100-point audit-prompt** — a runnable one-pass repo audit with point allocations, including a token-budget gate and `paths:`-coverage credit (source: https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/tools/audit-prompt.md).

---

## Sources

1. How Claude remembers your project — memory, rules, imports, enforcement, troubleshooting (official) — https://code.claude.com/docs/en/memory
2. Best practices for Claude Code — include/exclude table, prune test, emphasis, hooks-are-deterministic (official) — https://code.claude.com/docs/en/best-practices
3. Set up Claude Code in a monorepo or large codebase (official) — https://code.claude.com/docs/en/large-codebases
4. Understand context costs / features overview — per-feature cost, "request vs guarantee" (official) — https://code.claude.com/docs/en/features-overview
5. Explore the context window — /context, what survives compaction (official) — https://code.claude.com/docs/en/context-window
6. Debug your configuration — /memory, /context, /status, /doctor, clean-config bisection (official) — https://code.claude.com/docs/en/debug-your-config
7. Automate actions with hooks — PreToolUse deny, blocking semantics (official) — https://code.claude.com/docs/en/hooks-guide
8. Hooks reference — InstructionsLoaded payload, ConfigChange (official) — https://code.claude.com/docs/en/hooks
9. Set up Claude Code for your organization — managed settings, lockdown switches, /status source (official) — https://code.claude.com/docs/en/admin-setup
10. Settings reference — strictPluginOnlyCustomization, claudeMd, claudeMdExcludes (official) — https://code.claude.com/docs/en/settings
11. Monitoring usage — OTEL skill.name / plugin_loaded dead-config detection (official) — https://code.claude.com/docs/en/monitoring-usage
12. Security — prompt-injection threat model, ConfigChange audit hook (official) — https://code.claude.com/docs/en/security
13. How Claude Code works in large codebases (official blog) — https://claude.com/blog/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start
14. Prompting best practices — positive framing, explain-why, literalism (official) — https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
15. Evaluating AGENTS.md (ETH Zurich, arXiv 2602.11988) — context files reduce success, +20% cost (preprint) — https://arxiv.org/abs/2602.11988
16. Issue #5502 — CLAUDE.md adherence (ignore-until-asked), closed not-planned (official tracker) — https://github.com/anthropics/claude-code/issues/5502
17. Issue #38487 — path-scoped rules blind to file creation, closed not-planned (official tracker) — https://github.com/anthropics/claude-code/issues/38487
18. Issue #16853 — path-scoped rules not loaded in subdirs (official tracker) — https://github.com/anthropics/claude-code/issues/16853
19. Issue #16299 — path-scoped rules load globally regardless of paths: (official tracker) — https://github.com/anthropics/claude-code/issues/16299
20. Issue #37102 — automatic memory pruning, closed not-planned (official tracker) — https://github.com/anthropics/claude-code/issues/37102
21. Issue #38461 / #38493 — auto-dream unwired / unauditable (official tracker) — https://github.com/anthropics/claude-code/issues/38461
22. Issue #21674 — ~/.claude/CLAUDE.md writable injection vector, closed not-planned (official tracker) — https://github.com/anthropics/claude-code/issues/21674
23. agnix — linter/LSP/CI for CLAUDE.md/AGENTS.md/SKILL.md (community) — https://github.com/agent-sh/agnix
24. cclint (felixgeelhaar) — thresholds + deprecated-model check (community) — https://github.com/felixgeelhaar/cclint
25. cclint (carlrannaberg) — CLAUDE.md + hooks linter (community) — https://github.com/carlrannaberg/cclint
26. 50-point Team Adoption Scorecard (community) — https://www.digitalapplied.com/blog/claude-code-team-adoption-audit-50-point-scorecard-2026
27. 100-point audit-prompt (community) — https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/tools/audit-prompt.md
28. The pink-elephant negative-instruction analysis (community) — https://eval.16x.engineer/blog/the-pink-elephant-negative-instructions-llms-effectiveness-analysis
29. Writing a good CLAUDE.md — file:line over snippets, linter's-job (community) — https://www.humanlayer.dev/blog/writing-a-good-claude-md
30. Claude Code local memory security risk (community) — https://serendb.com/blog/claude-code-local-memory-security-risk
31. /context real token breakdown (community) — https://wmedia.es/en/tips/claude-code-context-command-token-usage
32. Modular rules naming/organization (community) — https://claude-blog.setec.rs/blog/claude-code-rules-directory

## Confidence & gaps

- **Verified official (high confidence):** every check whose basis cites `code.claude.com`, `claude.com/blog`, or `platform.claude.com` — the enforcement-vs-guidance split, the 200-line target, `@import` inline/4-hop behavior, the compaction-survival asymmetry, managed-policy mechanics, the debug commands, and the `InstructionsLoaded` payload were read from official docs on **2026-06-04**.
- **Official issue tracker (high confidence the report exists; status churns):** the path-scoped-rule bugs (#16853, #16299, #38487), the CLAUDE.md-adherence issue (#5502), the memory-pruning (#37102) and auto-dream (#38461/#38493) and injection (#21674) issues were open or closed-not-planned at fetch time. **Re-verify status against the running Claude Code version** before relying on a workaround — RULE-01/02/03 in particular are version-fragile.
- **Community / inference (medium confidence):** severity weights and line-count tiers (200–400 "ok", >1,000 "red flag") blend the official "<200 lines" target with community scorecards (digitalapplied, FlorianBruniaux) that are opinionated and slightly exceed the official anchor. Treat the official target as authoritative and the tiers as practitioner guidance.
- **Empirical (preprint):** the "auto-generated context files are net-negative / +20% cost" finding is from arXiv 2602.11988 (ETH Zurich preprint) plus secondary coverage; detailed sub-numbers are not from a peer-reviewed venue. It supports CONT-07/the bloat findings directionally, not as an Anthropic guarantee.
- **Static vs runtime (structural caveat):** an agent reading only a repo **cannot** confirm whether a file loads (`/memory`, `InstructionsLoaded`), what it costs (`/context`), whether managed policy applies (`/status`), or whether a skill/rule is dead (OTEL). Those checks are marked INFO and must be flagged "requires runtime verification," never scored as pass from a static read.
- **Version sensitivity:** auto memory (v2.1.59+), `managed-settings.d/` (v2.1.83+), and several lockdown keys are version-gated; a repo audited against an older CLI may legitimately lack them. Re-verify the feature set after major releases.

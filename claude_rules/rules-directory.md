# The `.claude/rules/` Directory

*CLAUDE.md decomposed into per-topic `.md` files — some always-on, some path-scoped; with reliability caveats an auditor must know. Compiled 2026-06-04.*

## Mental model

`.claude/rules/` is a directory of focused, single-topic `.md` files that together replace a monolithic CLAUDE.md. Two load modes exist, distinguished only by frontmatter: a rule **without** a `paths:` field loads at launch at the **same priority as `.claude/CLAUDE.md`**; a rule **with** `paths:` is conditional and is *meant* to load only when Claude reads a file matching its glob (source: https://code.claude.com/docs/en/memory). Like CLAUDE.md, rules are **guidance Claude reads, not configuration Claude Code enforces** — for guaranteed behavior use hooks or permissions (source: https://code.claude.com/docs/en/claude-directory). The load-priority and "context not enforcement" semantics are inherited wholesale from CLAUDE.md; see [claude-md-anatomy.md](claude-md-anatomy.md). The big real-world delta is **reliability**: path-scoped triggering has shipped broken in both directions across versions (never-loads and loads-globally), so an auditor must never assume a `paths:` rule actually loaded.

Auditor's one-liner: rules buy you per-topic decomposition and (when it works) deferred loading — but `paths:` is a soft optimization, not a guarantee; anything creation-critical or must-hold belongs in an unscoped rule, CLAUDE.md, or a hook.

## Setup, recursive discovery, and the canonical taxonomy

| Aspect | Mechanic | Source |
|---|---|---|
| Location | `.claude/rules/` (project); `~/.claude/rules/` (user-level, all projects) | (source: https://code.claude.com/docs/en/memory) |
| File type | Plain `.md`, one topic per file | (source: https://code.claude.com/docs/en/memory) |
| Discovery | **All** `.md` discovered **recursively** — subdirs like `frontend/`, `backend/` work with **no config** | (source: https://code.claude.com/docs/en/memory) |
| Canonical names | `code-style.md`, `testing.md`, `security.md`, `api-design.md` — descriptive single-topic filenames | (source: https://code.claude.com/docs/en/memory) |
| Naming rule | "Each file should cover one topic, with a descriptive filename" | (source: https://code.claude.com/docs/en/memory) |

- **Subdir organization is free.** Because discovery is recursive, group by tech layer (`frontend/`, `backend/`, `shared/`) or by feature domain (`payments/`, `analytics/`) at zero config cost (community, source: https://claude-blog.setec.rs/blog/claude-code-rules-directory). Audit signal: a flat pile of generically-named files (`rules1.md`, `stuff.md`, `frontend.md`) recreates the monolith problem inside the rules dir; flag it and recommend self-describing names (`api-validation.md`, not `frontend.md`) grouped into subdirs (community, source: https://claude-blog.setec.rs/blog/claude-code-rules-directory).
- **`--add-dir` does not load rules by default.** Rules from an additional directory load *only* when `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` is set, which then pulls `CLAUDE.md`, `.claude/CLAUDE.md`, `.claude/rules/*.md`, and `CLAUDE.local.md` from that dir (source: https://code.claude.com/docs/en/memory).

## Path-scoped rules: frontmatter, glob table, trigger semantics

A path-scoped rule carries a `paths:` frontmatter list. **Triggering semantics are precise: the rule loads when Claude *reads* a file matching the pattern — not on every tool use** (source: https://code.claude.com/docs/en/memory).

```markdown
---
paths:
  - "src/**/*.{ts,tsx}"
  - "tests/**/*.ts"
---
Use Zod for all runtime validation at module boundaries.
```

Officially supported glob forms:

| Pattern | Matches | Source |
|---|---|---|
| `**/*.ts` | All `.ts` files anywhere in the project | (source: https://code.claude.com/docs/en/memory) |
| `src/**/*` | Everything under `src/` (any depth) | (source: https://code.claude.com/docs/en/memory) |
| `*.md` | Files in the **project root only** (not nested) | (source: https://code.claude.com/docs/en/memory) |
| `src/components/*.tsx` | **Direct children** of `src/components/` only — not nested | (source: https://code.claude.com/docs/en/memory) |
| `src/**/*.{ts,tsx}` | Brace expansion — multiple extensions in one pattern | (source: https://code.claude.com/docs/en/memory) |
| multiple `paths:` entries | Listed patterns are OR'd together | (source: https://code.claude.com/docs/en/memory) |

- Brace expansion `{ts,tsx}` and multiple list entries are both supported (source: https://code.claude.com/docs/en/memory).
- `*.md` vs `src/components/*.tsx`: a single `*` does **not** cross directory boundaries; only `**` descends. Audit a `paths:` glob that intended deep matching but used single `*`.
- **Compaction behavior is undocumented.** No official source states whether a loaded path-scoped rule survives a `/compact`. By analogy to nested CLAUDE.md (re-injected only when Claude next reads a file in that subdir), a path-scoped rule *likely* needs a fresh matching-file read to reload post-compaction — **inference, not confirmed** (see [context-and-token-economics.md](context-and-token-economics.md) for what survives compaction).

## Priority model

| Layer | When it loads | Priority | Source |
|---|---|---|---|
| Unconditional rule (no `paths:`) | At launch | **== `.claude/CLAUDE.md`** | (source: https://code.claude.com/docs/en/memory) |
| Path-scoped rule (`paths:`) | When Claude reads a matching file | Conditional; applies only to matching work | (source: https://code.claude.com/docs/en/memory) |
| `~/.claude/rules/` (user-level) | Loaded **before** project rules | Lowest — **project wins** | (source: https://code.claude.com/docs/en/memory) |

- User-level rules apply to every project and load **before** project rules, "giving project rules higher priority" (source: https://code.claude.com/docs/en/memory). Reserve `~/.claude/rules/` for universal *personal* preferences that are fine to always load.
- Because an unconditional rule sits at `.claude/CLAUDE.md` priority, splitting a saturated CLAUDE.md into several unscoped rules **does not lower** any rule's priority — it distributes high-priority guidance across narrowly-named files so the model can tell what's relevant (source: https://code.claude.com/docs/en/memory).

## Reliability gotchas (the load-bearing section for auditors)

Path-scoped triggering is **version-fragile**. Three concrete failure modes, all backed by GitHub issues — treat each as a "verify on your running version" item:

| # | Failure mode | Reported version | Status | Source |
|---|---|---|---|---|
| #16853 | `paths:` rules in subdirs (e.g. `.claude/rules/api/`) **never auto-load** — not on read, not on edit, not even shown as available in `/context` | v2.1.1 | **OPEN** | (open GitHub issue, source: https://github.com/anthropics/claude-code/issues/16853) |
| #16299 | `paths:` rules load **globally** at session start regardless of `paths:` — context bloat (e.g. 28 rules loaded when ~5 should be) | v2.0.76 (flagged regression) | **OPEN** | (open GitHub issue, source: https://github.com/anthropics/claude-code/issues/16299) |
| #38487 | Even when working, scoping triggers via the pre-edit **Read** — so it loads on **edit** of an existing match but **not** on **Write** of a *new* file at a matching path; "blind to file creation — the exact moment when conventions matter most" | — | **closed not-planned** (opened 2026-03-25, closed 2026-04-30) | (GitHub issue, source: https://github.com/anthropics/claude-code/issues/38487) |

The two open bugs are **contradictory** (never-loads vs loads-everywhere) and were filed on different versions (v2.1.1 vs v2.0.76); which behavior is current on the latest release as of 2026-06-04 is unclear. Re-verify against the running version before relying on path-scoping.

**Audit posture (do this, in order):**
1. After launch, run `/memory` — it lists all loaded CLAUDE.md, CLAUDE.local.md **and rules** files, the canonical way to confirm a rule actually loaded (source: https://code.claude.com/docs/en/memory).
2. Run `/context` to see token cost per memory file (catches the #16299 loads-globally bloat) (source: https://code.claude.com/docs/en/memory).
3. Instrument the **`InstructionsLoaded` hook** — it logs exactly which instruction files loaded, when, and why, and is "useful for debugging path-specific rules or lazy-loaded files in subdirectories" (source: https://code.claude.com/docs/en/memory).
4. Keep **creation-critical / must-hold** rules **unscoped** (always resident) or behind a **PreToolUse hook / `permissions.deny`** — not a `paths:` rule (community, source: https://github.com/anthropics/claude-code/issues/38487). Example: "never write to `src/billing/` without validation" should be a hook on the Write tool, because the path-scoped rule is unguarded at exactly the creation moment.
5. **Don't** rely on path-scoping in `~/.claude/rules/` — user-level path-scoped rules have been reported silently ignored, and user rules are the lowest-priority layer anyway; keep path-scoped rules at **project** level (community, source: https://github.com/anthropics/claude-code/issues/16853).

The introduction version is community-asserted as **v2.0.64** ("ships native path matching"), but this is **not** confirmed in the official changelog reachable on 2026-06-04 (which exposes only 2.1.121+) — treat as community-tier (community, source: https://paddo.dev/blog/claude-rules-path-specific-native/; source: https://code.claude.com/docs/en/changelog).

## Symlinked shared rule libraries

`.claude/rules/` **supports symlinks** for sharing rule libraries across repos. Symlinks are resolved and loaded normally, and **circular symlinks are detected and handled gracefully**. You can symlink either a whole directory or a single file (source: https://code.claude.com/docs/en/memory):

```bash
ln -s ~/shared-claude-rules .claude/rules/shared          # whole directory
ln -s ~/company-standards/security.md .claude/rules/security.md   # single file
```

- **Pattern:** centralize org-wide standards in one repo/dir, symlink individual files into each project — update once, applies everywhere, with no copy-paste drift (source: https://code.claude.com/docs/en/memory).

## Cursor `.cursor/rules` comparison

Cursor shipped `.cursor/rules/` with glob-based path matching months before Claude Code (community, source: https://paddo.dev/blog/claude-rules-path-specific-native/). Key differences:

| Dimension | Cursor `.cursor/rules` | Claude Code `.claude/rules` |
|---|---|---|
| File extension | `.mdc` | plain `.md` |
| Glob frontmatter key | `globs:` | `paths:` |
| Activation modes | Four: **Always**, **Auto Attached**, **Agent Requested** (the three compared by paddo.dev) plus **Manual** = `@rule-name` (per Cursor's own docs); driven by an `alwaysApply` toggle + `globs:`/`description` | Two: unconditional (no `paths:`) vs path-scoped (`paths:`) |
| Rule-type system | Yes (`alwaysApply` / type) | No type system |

(community, source: https://paddo.dev/blog/claude-rules-path-specific-native/)

- **Interop nuance — read carefully:** Claude **does** honor its own `paths:` frontmatter; it does *not* "ignore frontmatter." What enables a single shared file to serve both tools is that **Claude ignores frontmatter keys it doesn't recognize** (e.g. Cursor's `globs:`, `alwaysApply`) while reading the body, and Cursor uses them — so one `.md`/`.mdc` file (or symlink) can drive both tools (community, source: https://paddo.dev/blog/claude-rules-path-specific-native/). Don't audit a file as "Claude ignores its frontmatter"; audit whether the `paths:` key is present and correct for Claude.

## Decision table: rule vs path-scoped rule vs nested CLAUDE.md vs @import vs skill

| Mechanism | Loads | Context cost | Use when | Source |
|---|---|---|---|---|
| **Unconditional rule** | At launch, `.claude/CLAUDE.md` priority | Always resident | Universal non-negotiable for *every* session (baseline style, security) | (source: https://code.claude.com/docs/en/memory) |
| **Path-scoped rule** | On reading a matching file (flaky — verify) | Deferred *when it works* | File-type/dir-specific conventions you've confirmed load on your version | (source: https://code.claude.com/docs/en/memory) |
| **Nested CLAUDE.md** | On demand when Claude reads files in that subdir | Deferred per-subtree | Per-directory context in a monorepo subtree | (source: https://code.claude.com/docs/en/memory) |
| **`@path` import** | **At launch, expanded inline** | **No saving** — costs same tokens | Organizing/splitting a long file *without* deferring load | (source: https://code.claude.com/docs/en/memory) |
| **Skill** | Only on invocation or when Claude deems it relevant | Lowest resident cost | Task-specific instructions that don't need to be always resident | (source: https://code.claude.com/docs/en/memory) |

- **Kill the @import myth:** `@path` imports "help organization but do not reduce context, since imported files load at launch." If the goal is to *defer* loading, use a path-scoped rule or a skill — **not** an `@import` (source: https://code.claude.com/docs/en/memory). See [claude-md-anatomy.md](claude-md-anatomy.md) for the @import 4-hop expansion detail.
- **Rule vs skill:** rules load every session (or when matching files open); "for task-specific instructions that don't need to be in context all the time, use [skills] instead, which only load when you invoke them or when Claude determines they're relevant" (source: https://code.claude.com/docs/en/memory).
- For the full three-axis routing (load-timing / context-cost / guarantee-vs-guidance) across *all* mechanisms, defer to [choosing-the-right-mechanism.md](choosing-the-right-mechanism.md). For grading a setup, see [audit-rubric.md](audit-rubric.md).

## `claudeMdExcludes` for rules dirs

`claudeMdExcludes` skips CLAUDE.md/rules by absolute glob — e.g. skip another team's entire rules tree in a monorepo (source: https://code.claude.com/docs/en/memory):

```json
"claudeMdExcludes": [
  "**/monorepo/CLAUDE.md",
  "/home/user/monorepo/other-team/.claude/rules/**"
]
```

- Arrays **merge** across settings layers; **managed-policy rules cannot be excluded** (source: https://code.claude.com/docs/en/memory). See [large-codebase-patterns.md](large-codebase-patterns.md) for monorepo exclusion strategy.

## Migrating a >200-line CLAUDE.md into rules

Migration is officially triggered: **"When CLAUDE.md approaches 200 lines, start splitting into rules"** (source: https://code.claude.com/docs/en/claude-directory). Recipe:

1. Decompose into `.claude/rules/{code-style,testing,security,api-design}.md`; keep CLAUDE.md as a thin overview/architecture file (source: https://code.claude.com/docs/en/memory).
2. Keep **cross-cutting non-negotiables** (security, baseline style) **unscoped** so they always load; move **file-type-specific** conventions behind `paths:` (source: https://code.claude.com/docs/en/memory).
3. Rationale — this solves **priority saturation**: when everything in one file is marked important, the model can't tell what's relevant; splitting distributes high-priority guidance across narrowly-scoped files while keeping universal constraints resident (source: https://code.claude.com/docs/en/memory).
4. After migrating, **verify with `/memory`** (rules actually loaded) and `/context` (no accidental global load of `paths:` rules per #16299).

CLAUDE.md size rationale (<200-line target, loaded in full regardless of length) lives in [claude-md-anatomy.md](claude-md-anatomy.md) and [context-and-token-economics.md](context-and-token-economics.md).

## Sources
1. https://code.claude.com/docs/en/memory — *(official)* — recursive discovery, `paths:` semantics, glob forms, priority, symlinks, taxonomy, `/memory`, `InstructionsLoaded`, `claudeMdExcludes`, `--add-dir`, @import-no-saving, rule-vs-skill.
2. https://code.claude.com/docs/en/claude-directory — *(official)* — rules are guidance not enforcement; 200-line migration trigger.
3. https://code.claude.com/docs/en/changelog — *(official)* — changelog coverage (used to note v2.0.64 is *not* present).
4. https://github.com/anthropics/claude-code/issues/16853 — *(community / open GitHub issue)* — never-loads-in-subdirs (v2.1.1).
5. https://github.com/anthropics/claude-code/issues/16299 — *(community / open GitHub issue)* — loads-globally regardless of `paths:` (v2.0.76).
6. https://github.com/anthropics/claude-code/issues/38487 — *(community / GitHub issue, closed not-planned 2026-04-30)* — blind to file creation (Read/Edit not Write).
7. https://paddo.dev/blog/claude-rules-path-specific-native/ — *(community)* — v2.0.64 attribution, Cursor `.cursor/rules` comparison, frontmatter interop.
8. https://claude-blog.setec.rs/blog/claude-code-rules-directory — *(community)* — subdir organization, generic-filename anti-pattern.
9. https://cursor.com/docs/context/rules — *(third-party / Cursor official docs)* — Cursor's four rule activation modes (Always Apply · Apply Intelligently = Agent Requested · Apply to Specific Files = Auto Attached · Apply Manually = `@rule-name`); confirms the fourth mode paddo.dev omits.

## Confidence & gaps
- **Verified official (safe as fact):** recursive `.md` discovery; unconditional-rule priority `== .claude/CLAUDE.md`; `paths:` triggers on *Read* of matching files; the glob table (`**/*.ts`, `src/**/*`, `*.md`, `src/components/*.tsx`, brace expansion, multiple entries); user-level loads before project (project wins); symlink support + graceful circular handling + the two `ln -s` recipes; rules are guidance not enforcement; the 200-line migration trigger; canonical filenames; `claudeMdExcludes` glob + array-merge + managed-can't-be-excluded; `--add-dir` gated on `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1`; `/memory`, `/context`, `InstructionsLoaded` as audit primitives; @import does not reduce context (loads at launch); rule-vs-skill guidance.
- **Community / version-sensitive (do not assert as Anthropic fact):** the three reliability bugs (#16853 never-loads, #16299 loads-globally, #38487 creation-blind) are real but version-fragile; the two open bugs are contradictory and filed on different versions (v2.1.1 vs v2.0.76), so current behavior on the latest 2026-06-04 release is unknown — **re-verify with `/memory`/`/context` on the running version.** The **v2.0.64** introduction version is community-asserted and **not** in the reachable official changelog (exposes only 2.1.121+). The frontmatter-interop nuance is community-tier (paddo.dev). The Cursor mode list is split-tier: the **three-mode** comparison (Always / Auto Attached / Agent Requested) is community-tier from paddo.dev, which names only three; the **fourth mode — Manual (`@rule-name`)** — is confirmed against Cursor's own docs (https://cursor.com/docs/context/rules).
- **Inference, explicitly flagged:** whether a path-scoped rule **survives compaction** is undocumented; by analogy to nested CLAUDE.md it likely needs a fresh matching-file read to reload post-`/compact` — not confirmed.
- **Unfetched primary sources:** some community references for this topic (e.g. claudelog.com FAQ, the Daniel San announcement tweet) returned 403/402 during research and were reachable only via search snippets — not cited here; nothing in this doc depends on them.

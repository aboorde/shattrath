# Large-Codebase & Monorepo Patterns

*Monorepo/large-repo architecture and org governance for Claude Code — what loads, who owns it, and what is actually enforced. Compiled 2026-06-04.*

In a monorepo a single root `CLAUDE.md` is an anti-pattern: it loads on every session everywhere, so a frontend task carries DB-migration rules and the context window fills with inapplicable instructions, inflating tokens and degrading adherence (source: https://code.claude.com/docs/en/large-codebases; source: https://thepromptshelf.dev/blog/claude-code-monorepo-setup/). The three governing levers are **two-level `CLAUDE.md` layering** (root = repo-wide gotchas + structure; per-package = local stack/commands, owned by each directory's owner), **starting-directory discipline** (the primary, zero-config scoping control), and **static exclusion** via `claudeMdExcludes` for packages you never touch. When per-directory layering stops scaling, a platform team migrates always-loaded conventions into versioned plugins/skills + an MCP code index, enforced org-wide via non-excludable managed policy and `managed-settings.d/` drop-ins (source: https://code.claude.com/docs/en/large-codebases). For the routing rubric (which mechanism for which need) see [choosing-the-right-mechanism.md](choosing-the-right-mechanism.md); for the scored checklist see [audit-rubric.md](audit-rubric.md).

## Two-level layering with distributed ownership

A single root `CLAUDE.md` in a large repo "tends to either grow to cover every subsystem's conventions, costing context on instructions unrelated to the current task, or stay too generic to be useful" (source: https://code.claude.com/docs/en/large-codebases). The documented fix is exactly two levels:

| Level | Holds | Owner | Size target |
|---|---|---|---|
| Root `./CLAUDE.md` | Repo-wide standards, commit conventions, layout/structure, critical cross-cutting gotchas | Platform team / root DRI | `<200` lines (source: https://code.claude.com/docs/en/memory) |
| Per-package / per-subsystem `CLAUDE.md` | That area's stack, build/test/lint commands, local conventions (one per package in a monorepo, one per subsystem in a single tree) | "Each directory's owner typically maintains its file" (source: https://code.claude.com/docs/en/large-codebases) | `<200` lines each |

- **Audit signal:** flag package-specific rules that have **leaked up** into the root file (an anti-pattern), and flag packages **missing** their own `CLAUDE.md`. If a rule applies to only one package it belongs in that package's file (community: https://thepromptshelf.dev/blog/claude-code-monorepo-setup/).
- Target **under 200 lines per file**; "longer files consume more context and reduce adherence," and `CLAUDE.md` is loaded in **full regardless of length** (source: https://code.claude.com/docs/en/memory). See [context-and-token-economics.md](context-and-token-economics.md) for the budget rationale.
- Community sizing heuristics (treat as estimates, not official): a 200–300 line root is "healthy"; beyond 500 lines "start questioning whether something belongs in a subdirectory" (community: https://thepromptshelf.dev/blog/claude-code-monorepo-setup/). Approximate working thresholds — ~30k lines where the agent loses a useful mental map, ~500k lines where LSP becomes necessary — are community estimates with no published methodology (community: https://claudefa.st/blog/guide/development/large-codebase-playbook).
- **Don't fake context reduction with `@imports`:** imported files are expanded and loaded at launch alongside the referencing file (max 4 hops). They help organization only, **not** context size (source: https://code.claude.com/docs/en/memory). To actually defer loading, use path-scoped `.claude/rules/` or skills — see [rules-directory.md](rules-directory.md).

## Starting-directory discipline (the primary scoping lever)

Where you start Claude determines file access, which `CLAUDE.md` loads at launch, **and** which project settings apply — more decisive and zero-config than any exclusion list (source: https://code.claude.com/docs/en/large-codebases).

| Start from | File access | `CLAUDE.md` loaded at launch | Sibling-package rules in context? |
|---|---|---|---|
| **Repo root** | Every file | Root `CLAUDE.md` only; subdir files load **on demand** when Claude reads files there | Yes, leaking in on demand as Claude reads across packages |
| **`packages/api/`** | That subtree only | `packages/api/CLAUDE.md` **plus every ancestor** (incl. root) — concatenated root→cwd | **No** — `packages/web/` instructions never enter context (source: https://code.claude.com/docs/en/large-codebases) |

- **Rule for auditors:** launch Claude inside the package/subsystem a task touches; reserve root-launch for genuinely cross-cutting work. This is more decisive than editing exclusion lists (source: https://code.claude.com/docs/en/large-codebases).
- **Project settings are NOT inherited up-tree.** `.claude/settings.json` loads **only from the starting directory** — "a `.claude/settings.json` at the repository root applies only when you start from the root" (source: https://code.claude.com/docs/en/large-codebases). Each subdirectory's `settings.json` must be **self-contained**; unlike `CLAUDE.md`, ancestor settings files do **not** stack.
- **Audit gotcha:** deny rules / worktree config placed only in a subdir's `settings.json` will not apply at root or inside worktrees — duplicate must-apply settings into the **root** `.claude/settings.json` (source: https://code.claude.com/docs/en/large-codebases).
- Scope build/test/lint commands per package in that package's file (e.g. `pnpm --filter @repo/api test`, `turbo run build --filter=@my-org/api`) so Claude doesn't run the full monorepo suite and waste context on unrelated build output (community: https://thepromptshelf.dev/blog/claude-code-monorepo-setup/).

## `claudeMdExcludes` — static exclusion semantics

`claudeMdExcludes` skips `CLAUDE.md` and rules files by path or glob (source: https://code.claude.com/docs/en/memory; source: https://code.claude.com/docs/en/large-codebases). Load-bearing mechanics an auditor should verify:

| Property | Behavior |
|---|---|
| Static vs per-task | "The exclusion list is static, not a per-task switch." To focus on a different package, **restart from that package's directory** instead of editing exclusions (source: https://code.claude.com/docs/en/large-codebases) |
| Path matching | Patterns match against **absolute** paths, so relative-style patterns must start with `**/` — forget the prefix and nothing matches (source: https://code.claude.com/docs/en/large-codebases) |
| Scope | Settable at user / project / local / managed scope |
| Merge | Arrays **merge** across scopes — a team sets project defaults; individuals add local overrides (source: https://code.claude.com/docs/en/memory) |
| Hard floor | **Managed policy `CLAUDE.md` can never be excluded** — org-wide instructions always apply regardless of individual settings (source: https://code.claude.com/docs/en/memory) |

Use `claudeMdExcludes` only for other teams' / legacy / vendored packages you never touch and must launch from root past; prefer subdir-launch for per-task focus.

## Reduce reads on vendored/generated code

For checked-in noise (vendored SDKs, committed generated code) add **`Read` deny rules** in `permissions.deny`. `.gitignore`'d paths (`node_modules`, `dist`) already stay out of search, so deny rules target *committed* noise (source: https://code.claude.com/docs/en/large-codebases). Caveats that bound their reliability:

- Deny rules cover built-in file tools and **recognized** Bash file commands (`cat`, `head`, `grep`, `find`) — but they **do not filter denied paths out of a recursive search's output**, and they **do not cover arbitrary subprocesses that open files themselves** (source: https://code.claude.com/docs/en/large-codebases). Treat them as a read-cost reducer, not a security boundary.
- `permissions.deny` arrays **merge** across all sources, so developers can extend managed deny lists but cannot remove from them (source: https://code.claude.com/docs/en/admin-setup) — making this a viable org-level read-reduction lever in managed settings.

## LSP / code-intelligence as a high-value investment

Claude navigates large codebases via **agentic search** (traversing the filesystem, reading files, following references) rather than a centralized RAG index, so "each developer's instance works from the live codebase" instead of a stale embedded snapshot that can't keep up with an active team (source: https://claude.com/blog/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start).

- LSP integration is called "one of the highest-value investments" for multi-language codebases (e.g. C/C++), giving symbol-level precision instead of text pattern-matching (source: https://claude.com/blog/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start).
- Blog extension-priority order for adding configuration: `CLAUDE.md` → Hooks → Skills → Plugins → **LSP integrations** → MCP servers → Subagents (source: https://claude.com/blog/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start). Community estimate: at ~500k lines grep "stops being a tool and becomes a tax" across vendored code, making LSP necessary (community: https://claudefa.st/blog/guide/development/large-codebase-playbook).

## Sparse worktrees for big monorepos

Pair `worktree.sparsePaths` with `symlinkDirectories` so worktrees (including subagent isolation) start faster and use far less disk (source: https://code.claude.com/docs/en/large-codebases):

- `worktree.sparsePaths` sparse-checks-out only the directories a task/subagent needs. **Always include `.claude`** so root settings/rules/skills reach the worktree (recall settings load only from the starting dir, and inside a worktree the working dir is the worktree root).
- `symlinkDirectories` symlinks `node_modules` back to the main checkout instead of duplicating it.
- `sparsePaths` are read from the **starting directory before the worktree is created** and apply to all worktrees in the session (source: https://code.claude.com/docs/en/large-codebases).

## `additionalDirectories` vs `--add-dir` loading matrix

Adding directories grants file access but, per the official table, does **not** auto-load their `CLAUDE.md`/rules the way starting inside them does — and the two mechanisms differ (source: https://code.claude.com/docs/en/large-codebases; source: https://code.claude.com/docs/en/memory):

| Mechanism | Set via | Grants file access | Loads `CLAUDE.md` + `.claude/rules/` | Loads skills |
|---|---|---|---|---|
| Start inside the dir | `cwd` | Yes (that subtree) | Yes — that dir + every ancestor, at launch | Yes |
| `--add-dir` / `/add-dir` | CLI flag / command | Yes | **Only** with `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` (then `CLAUDE.md`, `.claude/CLAUDE.md`, `.claude/rules/*.md`, `CLAUDE.local.md`) | **Yes** |
| `permissions.additionalDirectories` | `settings.json` | Yes | **Never** | **Never** |

Audit implication: granting a sibling package via `additionalDirectories` gives file access but **never** brings its conventions or skills into context; `--add-dir` brings skills but withholds `CLAUDE.md`/rules unless the env var is set. To actually load an area's instructions, **start Claude inside it**.

## Per-directory skills + discoverability at scale

Skills load on demand and are a primary "move it out of always-loaded `CLAUDE.md`" target (source: https://code.claude.com/docs/en/large-codebases). At monorepo scale the failure mode is **description starvation**:

- Starting from the **repository root**, skills from every subdirectory Claude touches during the session accumulate — "which can accumulate into the hundreds" — and descriptions get shortened when there are many, stripping the keywords Claude uses to pick a skill (source: https://code.claude.com/docs/en/large-codebases).
- **Mitigation:** keep skill descriptions short and **lead with words a request would actually contain** so selection survives truncation (source: https://code.claude.com/docs/en/large-codebases). See [choosing-the-right-mechanism.md](choosing-the-right-mechanism.md) for skill-vs-rule-vs-`CLAUDE.md` routing.

## The inflection point: layering → plugins / skills / MCP

"Per-directory `CLAUDE.md` files can become hard to govern as the codebase grows. Conventions drift, files go stale, and no one owns the root." Fixing this "falls to the team maintaining the Claude Code setup, not individual developers" (source: https://code.claude.com/docs/en/large-codebases). The documented escape hatch — move always-loaded content into load-on-demand, centrally-owned, versioned artifacts:

| From (always-loaded) | To (load on demand) | Why |
|---|---|---|
| Reference material in `CLAUDE.md` | **Skills** (loaded only when relevant) | Progressive disclosure cuts the always-on budget |
| Shared conventions needing version history / cross-repo reuse | **Plugins** (versioned bundles of skills/hooks/commands, owned centrally) | One place to version & update; `plugin-name:skill-name` namespace so plugin skills **never collide** with per-directory skills (source: https://code.claude.com/docs/en/large-codebases) |
| Reading files to learn structure | **MCP server** exposing an existing code-search/RAG index | Claude queries the index instead of reading files (source: https://code.claude.com/docs/en/large-codebases) |

A **`SessionStart` hook** can route engineers in unfamiliar areas: it reads the launch directory from hook input, looks it up in a committed path-to-plugin map, and prints the recommendation — "anything the hook prints to stdout is added to Claude's context before the first prompt" (source: https://code.claude.com/docs/en/large-codebases).

## Org governance

### Managed policy `CLAUDE.md` (non-excludable behavioral baseline)

Deployed at OS-specific paths via MDM / Group Policy / Ansible, or embedded directly with the `claudeMd` key (source: https://code.claude.com/docs/en/memory):

| OS | Managed policy `CLAUDE.md` path |
|---|---|
| macOS | `/Library/Application Support/ClaudeCode/CLAUDE.md` |
| Linux / WSL | `/etc/claude-code/CLAUDE.md` |
| Windows | `C:\Program Files\ClaudeCode\CLAUDE.md` |
| Any | `claudeMd` key inside `managed-settings.json` (content inline, no separate file) (source: https://code.claude.com/docs/en/memory) |

It loads **before** user and project `CLAUDE.md`, is honored **only** in managed/policy settings, and **cannot be excluded** (source: https://code.claude.com/docs/en/memory).

### `managed-settings.d/` drop-ins + merge order

A sibling drop-in directory lets independent teams deploy separate policy fragments (systemd convention): `managed-settings.json` merges first as base, then `*.json` files **sorted alphabetically** merge on top. Scalars — later wins; arrays — concatenated + de-duplicated; objects — deep-merged. Use numeric prefixes (`10-telemetry.json`, `20-security.json`) for deterministic order (source: https://code.claude.com/docs/en/settings).

### Settings precedence (high → low)

`Managed` > command-line args > `Local` (`.claude/settings.local.json`) > `Project` (`.claude/settings.json`) > `User` (`~/.claude/settings.json`). Array settings like `permissions.allow`/`permissions.deny` **merge** across all sources — developers can extend managed lists but **cannot remove** from them (source: https://code.claude.com/docs/en/admin-setup). Verify live with `/status`: the **"Enterprise managed settings"** line shows the source `(remote)`, `(plist)`, `(HKLM)`, `(HKCU)`, or `(file)`; server-managed settings reach devices at auth time and **refresh hourly during active sessions** (source: https://code.claude.com/docs/en/admin-setup).

### Enforcement vs guidance (the load-bearing distinction)

"Settings rules are enforced by the client regardless of what Claude decides to do. `CLAUDE.md` instructions shape Claude's behavior but are not a hard enforcement layer" (source: https://code.claude.com/docs/en/memory). So:

- **Hard enforcement** → managed settings: `permissions.deny`, `sandbox.enabled`, `env`, `forceLoginMethod` (source: https://code.claude.com/docs/en/memory).
- **Behavioral guidance** → managed `CLAUDE.md` (code style, compliance reminders) — **never** rely on any `CLAUDE.md` for security/compliance enforcement. See [choosing-the-right-mechanism.md](choosing-the-right-mechanism.md) for the guarantee-vs-guidance axis.

### Plugin-only lockdown: `strictPluginOnlyCustomization`

Managed-only switch that blocks skills, agents, hooks, and MCP servers from user/project sources so they come **only** from plugins or managed settings: `true` locks all four surfaces; an array (e.g. `["skills","hooks"]`) locks only the named ones — the central mechanism for plugin-only governance (source: https://code.claude.com/docs/en/settings).

### Roles, review cadence, and dead-skill detection

- **Ownership:** minimum viable is a single **DRI** with authority over configuration, permissions, the plugin marketplace, and `CLAUDE.md` conventions; larger orgs add an **"agent manager"** (hybrid PM/engineer). "Bottoms-up adoption generates enthusiasm but can fragment without someone to centralize what works" (source: https://claude.com/blog/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start).
- **PR-review `CLAUDE.md`:** "treat `CLAUDE.md` edits like any other documentation change so conventions track the code" (source: https://code.claude.com/docs/en/large-codebases).
- **Re-verification cadence:** "expect to do a meaningful configuration review every three to six months," and after major model releases — **delete rules that only worked around a now-fixed model limitation** (e.g. a rule forcing single-file refactors); old workaround rules become pure context overhead and reduce adherence (source: https://claude.com/blog/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start). See [adherence-and-anti-patterns.md](adherence-and-anti-patterns.md).
- **`Stop` hook (optional):** reviews the session transcript and proposes `CLAUDE.md` updates while the gap is fresh (source: https://code.claude.com/docs/en/large-codebases).
- **OTEL dead-skill detection:** enable the OTEL logs exporter with `OTEL_LOG_TOOL_DETAILS=1` so skill names are recorded **verbatim** instead of redacted; read `skill_activated` events' `skill.name` and `invocation_trigger` to find never-invoked skills to retire/consolidate (source: https://code.claude.com/docs/en/large-codebases). This keeps the selection list small so descriptions don't get truncated.

## Anti-patterns (flag these in an audit)

| Anti-pattern | Why it hurts | Fix |
|---|---|---|
| One monolithic root `CLAUDE.md` covering every subsystem | Loads everywhere; frontend work carries backend/DB rules; bloat reduces adherence (source: https://code.claude.com/docs/en/large-codebases) | Split root (`<200` lines, repo-wide only) + per-package files; launch from the relevant subdir |
| Assuming `@path` imports reduce context | Imported files expand and load at launch — organization only (source: https://code.claude.com/docs/en/memory) | Use path-scoped `.claude/rules/` or skills to defer; trim the file |
| Deny/worktree settings only in a subdir `settings.json` | Project settings load only from the starting dir; not inherited up-tree or into worktrees (source: https://code.claude.com/docs/en/large-codebases) | Duplicate must-apply settings in root `.claude/settings.json`; include `.claude` in `sparsePaths` |
| `claudeMdExcludes` used as a per-task focus switch | Static, not per-task; patterns match absolute paths (no `**/` = no match) (source: https://code.claude.com/docs/en/large-codebases) | Restart from the target package's dir; reserve excludes for never-touched packages |
| Relying on (managed) `CLAUDE.md` to enforce security/compliance | Delivered as a user message after the system prompt; not a hard enforcement layer (source: https://code.claude.com/docs/en/memory) | Put hard rules in managed settings / a `PreToolUse` hook |
| Per-directory `CLAUDE.md` proliferating with no root owner / no retirement | Drift, stale files, skill lists balloon into the hundreds (descriptions truncated → selection breaks) (source: https://code.claude.com/docs/en/large-codebases) | Assign a DRI; migrate stable conventions to plugins/skills; use OTEL `skill_activated` data to retire dead skills |

## Sources

1. https://code.claude.com/docs/en/large-codebases (official)
2. https://code.claude.com/docs/en/memory (official)
3. https://code.claude.com/docs/en/settings (official)
4. https://code.claude.com/docs/en/admin-setup (official)
5. https://claude.com/blog/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start (official)
6. https://thepromptshelf.dev/blog/claude-code-monorepo-setup/ (community)
7. https://claudefa.st/blog/guide/development/large-codebase-playbook (community)

## Confidence & gaps

- **Verified-official (state as fact):** two-level layering with per-directory ownership; starting-directory load behavior (root → root-only at launch + on-demand subdirs; subdir → that dir + all ancestors, no siblings); project `settings.json` loads only from the starting dir and is not inherited up-tree; `claudeMdExcludes` static/absolute-path/`**/`-prefix/array-merge/managed-non-excludable; `Read` deny-rule coverage and its search-output/subprocess caveats; `@import` non-deferral (4 hops); `<200`-line target loaded in full; managed policy `CLAUDE.md` paths + `claudeMd` key + non-excludability; `managed-settings.d/` merge order; settings precedence + array-merge; `strictPluginOnlyCustomization`; `plugin:skill` namespacing; OTEL `OTEL_LOG_TOOL_DETAILS=1` + `skill_activated`; `SessionStart`/`Stop` hooks; LSP "highest-value investment" + extension-priority order; DRI/agent-manager roles; 3–6-month review cadence; `/status` "Enterprise managed settings" line + hourly refresh (sources 1–5).
- **Community / inference (labeled in-line):** the 200–300/`>500`-line root sizing heuristics and ~30k/~500k-line thresholds (sources 6, 7) are practitioner estimates with **no published methodology** — do not present as Anthropic figures. Per-package command-scoping (`pnpm --filter`, `turbo`) is a community pattern (source 6).
- **Version-sensitive — re-verify after the next minor release:** settings keys `strictPluginOnlyCustomization`, `claudeMd`, and `enabledPlugins`, and OTEL event/attribute names. `managed-settings.d/` drop-ins were associated with ~v2.1.83+ in upstream ground truth but the fetched settings page did not restate that exact version — **re-confirm the introducing version against the changelog** before relying on it.
- **Out of scope / not documented:** no official endorsement of Nx/Turborepo/Bazel-specific integration (docs are tool-agnostic — substitute your own subsystem dir); community claims that Claude reads `nx.json`/`turbo.json` to infer the project graph are **not documented behavior**. The large-codebases blog is dated 2026-05-14; role definitions are described qualitatively — re-read before quoting verbatim.

# Choosing the Right Mechanism

*The decision system for placing any instruction across Claude Code's ~16 configuration mechanisms — routing by load-timing, context cost, and guarantee-vs-guidance. This is the hub the other docs link to. Compiled 2026-06-04.*

---

## Mental model: route by three axes

Every Claude Code mechanism answers an instruction with a different trade-off. Place anything by asking three questions in order:

1. **WHEN does it load?** — `always` (every request) / `path-matched` (only when a matching file is touched) / `on-demand` (only when invoked or judged relevant) / `event-driven` (fires on a tool call, prompt, or session event) / `enforced-by-client` (the harness applies it deterministically, no model involved).
2. **What is the CONTEXT COST?** — `every-request` (full text in the window each turn) / `cheap-until-used` (a description or tool name loads, body deferred) / `isolated` (runs in a separate window, only a summary returns) / `zero` (runs externally, costs nothing unless it returns output).
3. **Do you need a GUARANTEE or GUIDANCE?** — Prompt-level mechanisms (CLAUDE.md, `.claude/rules/`, `@imports`, skills, auto memory, output style, `--append-system-prompt`) are **guidance**: Claude can ignore, reconcile, or forget them. Only **hooks** and **`settings.permissions`** are **enforcement**: the client applies them regardless of what Claude decides (source: https://code.claude.com/docs/en/features-overview).

The single load-bearing rule: *an instruction is a request; a hook is a guarantee* (see [§3](#3-the-single-load-bearing-rule)). Default order: start at CLAUDE.md, promote to rules/skills as it grows, and escalate to hooks/permissions **only** when a rule must hold every single time. The promotion ladder ([§7](#7-the-promotion-ladder-build-your-setup-over-time)) is how an audited repo should have arrived at its current shape.

For applying this rubric **as a grader**, see [audit-rubric.md](audit-rubric.md). For the underlying cost numbers, see [context-and-token-economics.md](context-and-token-economics.md).

---

## 1. The master table

One row per mechanism. `Load timing` and `Context cost` use the vocabulary above. `G/g` = **G**uarantee (client-enforced) vs **g**uidance (model may ignore).

| Mechanism | Load timing | Context cost | G/g | What belongs there |
|---|---|---|---|---|
| **`CLAUDE.md`** | always (session start; re-read at root after compact) | every-request (full file) | g | Standing facts Claude should *always* know: conventions, build/test commands, project structure, "never do X" as rationale. Keep <200 lines (source: https://code.claude.com/docs/en/memory). |
| **`.claude/rules/` (no `paths:`)** | always (session start) | every-request | g | Same priority as `.claude/CLAUDE.md`; use to keep CLAUDE.md focused by topic-splitting always-on rules into separate files (source: https://code.claude.com/docs/en/features-overview). |
| **`.claude/rules/` (with `paths:`)** | path-matched (loads when Claude reads a glob-matching file) | cheap-until-matched | g | Path-specific guidelines (e.g. "in `**/*.tsx` use X"). Saves context — only loads when working with matching files (source: https://code.claude.com/docs/en/features-overview). **Version-fragile** ([§4](#path-scoped-fragility-audit-trap)). |
| **Nested `CLAUDE.md`** (subdirectory) | on-demand (loads when Claude reads files in that dir) | every-request *once loaded* | g | Directory-local conventions in a monorepo. **Not re-injected after `/compact`** until Claude next reads a file there (source: https://code.claude.com/docs/en/hooks-guide). |
| **`@path` import** | always (expanded inline at launch, max 4 hops) | every-request — **does NOT defer cost** | g | Organization only: split a long CLAUDE.md into files. Imported content is loaded in full at launch alongside the referencing file (source: https://code.claude.com/docs/en/memory). |
| **Skill** | on-demand (description loads at start; body when invoked/relevant) | cheap-until-used (description every-request) | g | Reference material needed *sometimes* (API docs, style guides) or a `/<name>` workflow (deploy, review, release) (source: https://code.claude.com/docs/en/features-overview). |
| **Skill + `disable-model-invocation: true`** | manual-only (hidden from Claude until you type `/<name>`) | **zero until invoked** | g | Side-effect/manual skills (`/deploy`, `/commit`) you alone should trigger (source: https://code.claude.com/docs/en/skills). |
| **Hook — `PreToolUse`** | event-driven (before a tool call; **can block**) | zero unless it returns output | **G** | Hard guardrails: block an edit/command before it runs. Survives `bypassPermissions` (source: https://code.claude.com/docs/en/hooks-guide). |
| **Hook — `PostToolUse` / `SessionStart` / etc.** | event-driven (after the fact; **cannot undo**) | zero unless it returns output | partial | Side effects: lint/format/log after an edit; re-inject context after `/compact` (source: https://code.claude.com/docs/en/hooks-guide). |
| **`settings.permissions.deny`** | enforced-by-client (every matching tool call) | zero | **G** | Deny rules for tools/paths (e.g. `Read(./.env)`). Merges across scopes; **deny always wins**, even over a hook `allow` (source: https://code.claude.com/docs/en/settings). |
| **Subagent** | on-demand (spawned for a task) | isolated (own window; only summary returns) | g | Context-isolated heavy reads, parallelism, cheaper-model mechanical work, tool-locked-down tasks (source: https://code.claude.com/docs/en/features-overview). |
| **Agent team** | on-demand (experimental, off by default) | isolated × N (each a separate Claude instance) | g | When parallel workers must *message each other* and share a task list — higher cost than subagents (source: https://code.claude.com/docs/en/features-overview). |
| **Plugin** | packaging (bundles skills/agents/hooks/MCP/LSP + default `settings.json`) | inherits whatever it bundles | varies | Share the same setup across repos/teams, version it, or distribute via marketplace. Plugin skills are namespaced `/plugin:skill` (source: https://code.claude.com/docs/en/plugins). |
| **MCP server** | on-demand (tool names + server instructions at start; schemas deferred) | cheap-until-used (Tool Search on by default) | g | Connect to an external system — the protocol/connection providing tools and data, auth handled by the server (source: https://code.claude.com/docs/en/mcp). |
| **Auto memory** (Claude-written) | always (MEMORY.md index: first 200 lines / 25KB) | every-request (index) + on-demand (topic files) | g | *Learned* patterns Claude discovers — the inverse of CLAUDE.md (which *you* write). Requires v2.1.59+ (source: https://code.claude.com/docs/en/memory). See [auto-memory-and-self-improvement.md](auto-memory-and-self-improvement.md). |
| **Output style** | always (read once at session start; adjusts the system prompt) | every-request (system-prompt level) | g | Persona/tone/format shaping of the whole session. Confirmed via the settings page (`outputStyle`) (source: https://code.claude.com/docs/en/settings). |
| **`--append-system-prompt`** | always (this session only) | every-request (system-prompt level) | g | Ephemeral per-invocation system-prompt addendum for scripted/CI runs; not persisted to a file. |

---

## 2. The context-cost ground truth

The official "context cost by feature" table is the basis for the `Context cost` column above (source: https://code.claude.com/docs/en/features-overview):

- **CLAUDE.md** — loads full content at session start; costs context *every request*.
- **Skills** — descriptions load at start; full body loads only when used. Low recurring cost, but descriptions share a budget (~1% of the context window, capped per entry) and silently truncate when many skills are loaded (source: https://code.claude.com/docs/en/skills).
- **MCP** — tool names + server instructions load at start; full schemas deferred until needed by Tool Search (on by default), so adding servers has minimal impact (source: https://code.claude.com/docs/en/mcp).
- **Subagents** — isolated from the main session; the parent only receives a summary.
- **Hooks** — load nothing and cost zero unless the hook returns additional context (source: https://code.claude.com/docs/en/features-overview).

For real `/context` numbers, what survives compaction, and the @import "no-deferral" myth in depth, defer to [context-and-token-economics.md](context-and-token-economics.md).

---

## 3. The single load-bearing rule

> *Put guardrails in hooks. An instruction like "never edit `.env`" in CLAUDE.md or a skill is a request, not a guarantee. A `PreToolUse` hook that blocks the edit is enforcement. If a rule must hold every time, make it a hook rather than a prompt instruction.* (source: https://code.claude.com/docs/en/features-overview)

This is the highest-yield audit check. CLAUDE.md is delivered as a **user message after the system prompt** and is treated as **context, not enforced configuration** — Claude can reconcile or ignore it (source: https://code.claude.com/docs/en/memory). So:

- **"never edit .env" in prose is NOT enforcement.** Back it with a `PreToolUse` hook (`exit 2` / `permissionDecision: "deny"`) or a `permissions.deny` rule.
- **`PreToolUse` deny is the strongest guardrail in the system.** It fires before any permission-mode check and blocks the tool **even in `bypassPermissions` mode or with `--dangerously-skip-permissions`** — policy users cannot bypass by switching modes (source: https://code.claude.com/docs/en/hooks-guide).
- **Managed deny outranks a hook allow.** Hooks can *tighten* but never *loosen* past permission rules; a hook returning `allow` never overrides a `settings` deny (source: https://code.claude.com/docs/en/hooks-guide). Settings precedence, highest first: **Managed/enterprise > CLI args > local `.claude/settings.local.json` > project `.claude/settings.json` > user `~/.claude/settings.json`** — managed can't be overridden by anything (source: https://code.claude.com/docs/en/settings).

**Escalate, don't duplicate.** Write the human-readable rule once in CLAUDE.md as *explanation*; if it MUST hold every time, add the hook/`permissions.deny`. For org-wide non-bypassable policy, put the deny rule (plus `forceLoginMethod` / `strictPluginOnlyCustomization` / `allowManagedPermissionRulesOnly`) in **managed settings** (source: https://code.claude.com/docs/en/settings). The `claudeMd` key injects org memory but is only honored in managed/policy settings.

---

## 4. Hook event taxonomy — what can actually block

Only a subset of hook events can *prevent* an action; the rest can only react. Routing a "must-block" requirement to the wrong event is a silent failure (source: https://code.claude.com/docs/en/hooks-guide).

| Event | Can block? | Use it for |
|---|---|---|
| `PreToolUse` | **Yes** — `exit 2` (stderr → feedback) or `permissionDecision: "deny"` | Blocking a tool call before it runs (edits, Bash, MCP writes). |
| `UserPromptSubmit` / `UserPromptExpansion` | **Yes** — can block the prompt / its expansion | Reject or rewrite a user prompt before Claude acts on it. |
| `PostToolUse` | **No** — fires after success; **cannot undo** | Side effects only: lint, format, log, append feedback. |
| `SessionStart` / `Setup` / `Notification` | **No** — `exit 2` only surfaces stderr | Re-inject context, scaffolding, alerts. |

> *Exit 2: the action is blocked. Write a reason to stderr, and Claude receives it as feedback so it can adjust.* (source: https://code.claude.com/docs/en/hooks-guide)

**Anti-pattern:** relying on a `PostToolUse` (or any post-event) hook to "block" an action. It fires after the tool already executed and cannot reverse it. Use `PreToolUse`/`UserPromptSubmit` for blocking; reserve `PostToolUse` for side effects (source: https://code.claude.com/docs/en/hooks-guide).

**Judgment calls (not deterministic rules)** route to the two non-script hook types (source: https://code.claude.com/docs/en/hooks-guide):

- **`type: prompt`** — a single LLM call (Haiku default). *Use when the hook input data alone is enough to decide.*
- **`type: agent`** — can read files and run commands, up to 50 turns. *Use when you need to verify against the actual state of the codebase.*

<a id="path-scoped-fragility-audit-trap"></a>
**Compact-survival asymmetry (audit trap):** after `/compact`, the project-root CLAUDE.md is re-read, but a **nested subdirectory CLAUDE.md is NOT re-injected** until Claude next reads a file there. Use a `SessionStart` hook with `matcher: compact` to re-inject critical conventions, and the `InstructionsLoaded` hook to log which instruction files actually loaded — closing the gap between *what you wrote* and *what is in context* (source: https://code.claude.com/docs/en/hooks-guide). Pair with `/memory` to audit the live set. Note: path-scoped `.claude/rules/` are **version-fragile** — see [rules-directory.md](rules-directory.md) for issues #16853 (never loads in subdirs), #16299 (loads globally ignoring `paths:`), and #38487 (blind to file *creation* — triggers on Read/Edit not Write).

---

## 5. The isolation/cost spectrum: skill → subagent → agent team, and MCP vs skill

**Skill vs Subagent vs Agent team** form a single axis from *low cost / shared context* to *high isolation / separate process* (community synthesis: https://boringbot.substack.com/p/claude-code-skills-subagents-hooks):

| | Context | Cost | Communication |
|---|---|---|---|
| **Skill** | Adds to your main window; shares content across contexts | Lowest | N/A (inline) |
| **Subagent** | Separate window; own input/output tokens; only a **summary** returns | Medium | Reports back to the main agent only |
| **Agent team** | Each teammate is a **separate Claude instance** | Highest | Teammates message each other directly + shared task list |

- *Use a subagent when you need context isolation or when your context window is getting full. The subagent might read dozens of files or run extensive searches, but your main conversation only receives a summary* (source: https://code.claude.com/docs/en/features-overview).
- *Use an agent team when teammates need to share findings, challenge each other, and coordinate independently* — transition to teams only when parallel subagents hit context limits or need to communicate (source: https://code.claude.com/docs/en/features-overview). Teams are **experimental and disabled by default**, so this is provisional advice.
- **Subagent startup load:** its own system prompt (not the full Claude Code system prompt), the full content of any skills in its `skills:` field, plus CLAUDE.md and git status — **except** the built-in Explore and Plan agents omit both. Subagents do **not** inherit your conversation history or invoked skills (source: https://code.claude.com/docs/en/sub-agents). Scope precedence: managed > `--agents` CLI flag > project > user > plugin.

**Community signals for subagent vs skill** (on top of the official isolation rule) (source: https://theaiarchitects.com/blog/claude-code-subagents-vs-skills): reach for a **subagent** when work (1) reads many files / would flood context, (2) is mechanical (lint/format/validate) and can run on a cheaper pinned model, or (3) touches sensitive tools you must lock down with `tools`/`disallowedTools`. Otherwise **write a skill first** — easier to author, faster to debug, cheaper to run, shares parent context.

**MCP vs Skill** is *connection* vs *knowledge*, and they combine (source: https://code.claude.com/docs/en/features-overview):

> *MCP gives Claude purpose-built tools for an external system, with the connection and authentication handled by the server. Skills give Claude knowledge about how to use those tools effectively.*

To keep a write-heavy or rarely-needed MCP server **out of the main conversation entirely**, define it inline in a subagent's `mcpServers` frontmatter rather than in `.mcp.json` — the subagent gets the tools, the parent does not (source: https://code.claude.com/docs/en/sub-agents). MCP scope precedence, highest first: **local > project (`.mcp.json`, version-controlled) > user > plugin-provided > claude.ai connectors**; the entire server entry from the highest source wins (fields are *not* merged).

---

## 6. How same-feature layers reconcile

When the same kind of mechanism is defined at multiple scopes, the reconciliation rule differs by mechanism (source: https://code.claude.com/docs/en/features-overview):

| Mechanism | Reconciliation |
|---|---|
| **CLAUDE.md / rules** | **Additive** — all levels (managed → user → project → local) contribute simultaneously; conflicts reconciled by Claude's judgment, more-specific usually wins. |
| **Skills / Subagents** | **Override by name** — a same-named definition from a higher-precedence scope replaces the lower one. |
| **MCP** | **Override by name** — the whole server entry from the highest source wins; fields not merged. |
| **Hooks** | **Merge** — *all registered hooks fire for their matching events regardless of source.* |
| **Permissions** | **Merge across scopes; deny always wins** (managed deny is non-bypassable). |

Implication for auditors: you cannot "turn off" a managed hook or a managed deny from a lower scope. And because hooks merge, a project hook and a user hook both fire — there is no silent shadowing.

---

## 7. The promotion ladder (build your setup over time)

The official "build your setup over time" triggers give a concrete escalation path — a repo's mechanism choices should trace back to one of these (source: https://code.claude.com/docs/en/features-overview):

| Trigger | Promote to |
|---|---|
| Claude got a convention **wrong twice** | Add a line to **CLAUDE.md** |
| You **retyped the same prompt** | A **user-invocable skill** |
| You **pasted the same playbook a 3rd time** | A **skill** |
| You keep **copying from a browser tab** | An **MCP** server |
| A side task **floods the conversation** | A **subagent** |
| You want it to **happen every time without asking** | A **hook** |
| A **second repo** needs the same setup | A **plugin** |

> *You want something to happen every time without asking → Write a hook* (source: https://code.claude.com/docs/en/features-overview)

**Keep orchestration in the harness, not in a subagent.** Put routing/enforcement in hooks/settings (deterministic, client-applied); reserve subagents as single-purpose isolated workers. Mixing the task-isolation layer with the orchestration layer is the root cause of unauditable "my agent setup is a mess" situations (community: https://boringbot.substack.com/p/claude-code-skills-subagents-hooks).

**Right-size skill cost via frontmatter** instead of deleting skills (source: https://code.claude.com/docs/en/skills): default (description always loaded) for auto-pickable skills; `disable-model-invocation: true` for side-effect/manual skills (zero context, `/`-only); `user-invocable: false` for background-knowledge skills (hidden from the `/` menu, still model-invocable); `paths:` to scope auto-activation; and demote un-editable third-party skills via `skillOverrides` in settings. Run `/doctor` to see description-budget overflow and `/mcp` for per-server token cost.

---

## 8. The routing rubric (one paragraph)

To place any instruction, walk the axes in priority order: if it **must be guaranteed**, route to a `PreToolUse` hook (`exit 2` / `permissionDecision: deny`) or `permissions.deny` — and to managed settings if the team cannot be allowed to bypass it; if it is an **always-on behavioral fact**, put it in CLAUDE.md (or an unconditional `.claude/rules/` file) and keep CLAUDE.md under 200 lines; if it is a **path-scoped guideline**, use a `.claude/rules/` file with `paths:` frontmatter (accepting its version fragility); if it is an **on-demand procedure or reference**, write a skill (use `disable-model-invocation` for side-effect/manual ones); if it is a **context-isolated heavy read or parallel/mechanical job**, spawn a subagent (escalate to an agent team only when workers must talk to each other); if it needs **external data or actions**, connect an MCP server (and inline it in a subagent to keep its schemas off the main window); if it is a **fact Claude learned**, let auto memory record it; and if you need the same setup in a **second repo**, package it as a plugin. The default bias is always to start at CLAUDE.md and promote upward via the ladder in [§7](#7-the-promotion-ladder-build-your-setup-over-time) — escalating to enforcement only when guidance has demonstrably failed. To grade a repo against this rubric, hand off to [audit-rubric.md](audit-rubric.md).

---

## Sources

1. https://code.claude.com/docs/en/features-overview — *(official)* Claude Code "Features overview." Backs: the match-features-to-goal matrix, the compare-similar-features tables (CLAUDE.md vs skill, skill vs subagent, subagent vs agent team, MCP vs skill), the per-feature context-cost table, the same-feature reconciliation rules (additive / override-by-name / merge), the "request vs guarantee" guardrail rule, and the build-your-setup-over-time promotion ladder.
2. https://code.claude.com/docs/en/memory — *(official)* "Memory." Backs: CLAUDE.md delivered as a user message after the system prompt and treated as context not enforced config; `@path` imports expanded inline at launch (max 4 hops, no context deferral); auto memory MEMORY.md index (first 200 lines / 25KB), v2.1.59+; <200-line target.
3. https://code.claude.com/docs/en/hooks-guide — *(official)* "Hooks guide." Backs: the blocking event taxonomy (`PreToolUse`/`UserPromptSubmit` can block, `PostToolUse` cannot undo, `SessionStart`/`Notification` cannot block), `exit 2` / `permissionDecision: deny` semantics, `PreToolUse` deny surviving `bypassPermissions`, hooks-tighten-not-loosen, `type: prompt` vs `type: agent`, and the compact-survival asymmetry + `InstructionsLoaded`/`SessionStart` re-injection.
4. https://code.claude.com/docs/en/settings — *(official)* "Settings." Backs: settings precedence (managed highest, can't be overridden), permission-rule merge with deny-wins, the managed-only lockdown switches (`strictPluginOnlyCustomization`, `allowManagedPermissionRulesOnly`, `forceLoginMethod`), the `claudeMd` org-memory key, `outputStyle`, and `skillOverrides`.
5. https://code.claude.com/docs/en/skills — *(official)* "Skills." Backs: `disable-model-invocation: true` (zero cost until invoked), `user-invocable: false`, `paths:` scoping, the description-budget (~1% / per-entry cap) silent-truncation risk, and `/doctor` budget overflow.
6. https://code.claude.com/docs/en/sub-agents — *(official)* "Subagents." Backs: subagent startup load (own system prompt, `skills:` field, CLAUDE.md/git status except Explore/Plan), no inheritance of history/invoked skills, scope precedence, and the inline `mcpServers` context-routing technique.
7. https://code.claude.com/docs/en/mcp — *(official)* "MCP." Backs: MCP scope precedence (local > project > user > plugin > connectors), entire-entry-wins (no field merge), and Tool Search deferring schemas so idle servers cost little.
8. https://code.claude.com/docs/en/plugins — *(official)* "Plugins." Backs: plugin as a packaging layer bundling skills/agents/hooks/MCP/LSP + default settings, namespaced `/plugin:skill`, and when to use a plugin vs standalone `.claude/`.
9. https://theaiarchitects.com/blog/claude-code-subagents-vs-skills — *(community)* Backs the three subagent-vs-skill signals (floods context / mechanical cheaper-model / sensitive tools) and the "write a skill first" default.
10. https://boringbot.substack.com/p/claude-code-skills-subagents-hooks — *(community)* Backs the skill→subagent→agent-team cost-vs-isolation spectrum and the harness-vs-subagent orchestration separation.

## Confidence & gaps

- **Verified-official (high confidence):** the entire master table's load-timing / context-cost / guarantee-vs-guidance assignments, the "request vs guarantee" rule, the hook blocking taxonomy, `PreToolUse` deny surviving `bypassPermissions`, managed-deny > hook-allow precedence, the same-feature reconciliation rules, the promotion ladder, and all MCP/subagent/skill scoping. These come directly from `code.claude.com/docs` pages and are safe to state as fact.
- **Synthesis / inference (not an official single statement):** the **cost-vs-isolation spectrum** (skill → subagent → agent team) and the **harness-vs-subagent orchestration** framing are community-sourced (boringbot, theaiarchitects); the official docs *imply* but never state these as one axis. Treat them as a useful lens, not an Anthropic guarantee. The three-axis routing rubric itself is this doc's synthesis over the official tables.
- **Version-sensitive (re-verify on an older CLI):** auto memory (v2.1.59+), and several routing levers noted across the doc set are version-gated — managed-settings.d (v2.1.83+), the hook `if` field (v2.1.85+), `alwaysLoad` (v2.1.121), `--plugin-dir .zip` (v2.1.128). A repo audited on an earlier CLI may lack them. Skill description-budget figures (~1% of context window, per-entry char cap) are current as of 2026-06-04 but are model/version-tunable (`skillListingBudgetFraction`, `maxSkillDescriptionChars`).
- **Provisional / lightly-documented:** **agent teams** are experimental and disabled by default, so "escalate to a team" is conditional advice. **Output style** and **`--append-system-prompt`** were confirmed at the system-prompt level (the settings page documents `outputStyle` as read once at session start); a dedicated output-styles page was not deep-fetched in this pass, so their finer behavior is asserted at the settings level only.
- **Path-scoped `.claude/rules/` fragility** is flagged here but documented in detail (with the specific GitHub issues) in [rules-directory.md](rules-directory.md); the version at which rules shipped is community-asserted, not in the reachable official changelog.

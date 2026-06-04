# CLAUDE.md & .claude/rules/ — Power-User Cheat-Sheet

*One-page field reference distilled from the sourced docs in this folder — citations live there; this page is the gist. Compiled 2026-06-04; re-check version-gated and community items against the live docs.*

---

### Mental model
**CLAUDE.md is a finite, lossy instruction budget — context, not enforcement.** It is delivered as a *user message after the system prompt* with no compliance guarantee, loaded **in full every session**. Adherence decays with instruction count and conversation length. Keep only non-inferable, behavior-shifting facts; push everything else to the mechanism that fits; and migrate anything that *must hold* out of prose into a hook/permission. **Rules in prompts are requests; hooks in code are laws.**

### Route by three axes
**WHEN it loads** (always / path-matched / on-demand / event-driven / enforced-by-client) · **CONTEXT COST** (every-request / cheap-until-used / isolated / zero) · **GUARANTEE vs GUIDANCE** (only hooks + `permissions` are guarantees).

| Need | Put it in | Loads | Guarantee? |
|---|---|---|---|
| Always-on behavioral fact | `CLAUDE.md` / unconditional `.claude/rules/*.md` | every request | no |
| File-type / dir-specific guideline | `.claude/rules/` **with `paths:`** (verify it loads) | on matching-file read | no |
| Per-directory context (monorepo) | nested `CLAUDE.md` | on demand in that subtree | no |
| On-demand procedure / reference | **skill** (`disable-model-invocation` for side-effect ones) | description always, body when used | no |
| Heavy reads / parallel / mechanical | **subagent** | isolated; summary returns | no |
| External data / actions | **MCP** | names at start, schemas deferred | no |
| Learned fact | **auto memory** (then promote up to share) | MEMORY.md index every session | no |
| **Must hold every time** | **`PreToolUse` hook / `permissions.deny`** | event / enforced | **YES** |
| Same setup in another repo | **plugin** | bundles the above | — |

`@imports` are **not** a deferral lever — they expand inline at launch (4-hop max) and cost the same tokens.

### The numbers that matter
- **`CLAUDE.md` target: <200 lines** (loaded in full regardless; longer = more tokens + *less adherence*).
- **`@import` depth: 4 hops max**; expanded inline at launch.
- **Auto memory `MEMORY.md`: first 200 lines OR 25KB** load each session (the rest silently doesn't); auto memory needs **v2.1.59+**.
- **Skill body after `/compact`: capped 5,000 tokens/skill, 25,000 total** (kept from the top → front-load critical steps).
- **Empirical:** auto-generated/bloated context files can *reduce* success and add **>20% cost** (ETH Zurich, arXiv 2602.11988); instruction-following collapses to ~62–69% at 500 simultaneous instructions (IFScale, arXiv 2507.11538).

### Enforcement hierarchy (hardest → softest)
**managed `permissions.deny`** > **hook `deny`** (survives `bypassPermissions`/`--dangerously-skip-permissions`) > hook `allow` > **`CLAUDE.md`/rules guidance**. Deny rules from *any* scope outrank a hook allow. Only `PreToolUse`/`UserPromptSubmit` can **block**; `PostToolUse` fires too late to undo.

### What survives `/compact`
| Re-injected from disk | Lost until re-triggered |
|---|---|
| Project-root `CLAUDE.md` + **unscoped** rules; auto memory | Path-scoped (`paths:`) rules; **nested** subdir `CLAUDE.md`; the **skill-description index** |

→ Anything that must persist belongs in **root CLAUDE.md or an unscoped rule** (or a hook).

### Authoring a kept line
- **Prune test (official):** *"Would removing this cause Claude to make mistakes? If not, cut it."* Bloat makes Claude ignore the rules that matter.
- **INCLUDE:** non-guessable commands, deviations from defaults, test runners, repo etiquette, architecture decisions, gotchas. **EXCLUDE:** anything inferable from code, standard conventions, API docs (link), file-by-file trees, "write clean code".
- **Positive > negative** ("use named exports" beats "don't use default exports"); **attach the why**; **state scope explicitly** (Opus interprets literally, won't generalize); **show one positive example**.
- **Emphasis is a scarce signal** — `IMPORTANT`/`YOU MUST` on 1–2 rules only; if everything's important, nothing is. A persistently-ignored rule means the *file is too long*, not under-emphasized.
- **No snippets / pasted docs** — use `file:line` refs (they go stale + cost budget). `<!-- HTML comments -->` are stripped before injection → free maintainer notes.
- Never commit `/init` output verbatim — treat it as a draft and prune.

### `.claude/rules/` essentials
- All `.md` discovered **recursively** (subdirs free); one topic per file, descriptive names (`testing.md`, not `stuff.md`).
- Unconditional rule = `.claude/CLAUDE.md` priority/cost. `~/.claude/rules/` = user-level, loads *before* project (project wins).
- **Symlinks** supported → shared org rule libraries; circular handled.
- **Reliability caution:** `paths:` triggering is version-fragile — broken both ways (#16853 never-loads, #16299 loads-globally) and **blind to file creation** (#38487, loads on edit not Write). Keep creation-critical / must-hold rules **unscoped** or in a hook; verify with `/memory`/`/context`.
- Migrate at ~200 lines: split into `rules/{code-style,testing,security,api-design}.md`, keep cross-cutting non-negotiables unscoped.

### Monorepo / large repo
- **One root CLAUDE.md = anti-pattern.** Two levels: root (repo-wide, <200 lines) + per-package (owned by directory owners).
- **Starting-directory discipline is the primary scoping lever:** launch inside the package a task touches (loads that dir + ancestors, no siblings). `settings.json` is **not** inherited up-tree.
- `claudeMdExcludes` (static, absolute glob, `**/` prefix, arrays merge; managed never excluded). `additionalDirectories` grants access but **never** loads CLAUDE.md/rules/skills; `--add-dir` loads skills (and CLAUDE.md/rules only with `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1`).
- When layering stops scaling → centralize into **plugins/skills + an MCP code index**, lock down via `strictPluginOnlyCustomization`. Assign a **DRI**; PR-review CLAUDE.md; re-verify every 3–6 months (delete stale model-workaround rules). Find dead skills via OTEL `skill_activated`.

### Auto memory (the Claude-written sibling)
- `~/.claude/projects/<project>/memory/`, **machine-local**, **manual-prune only** (no auto-prune — #37102 not-planned). A wrong learning loads **every session until a human deletes it** → audit it like a dependency via `/memory`.
- Keep `MEMORY.md` a thin index. To **share** a learning, promote it into version-controlled CLAUDE.md/rules. Subagent `memory: project` is the one auto-written memory you *can* git-track.
- Don't confuse three "memory" systems: built-in auto memory ≠ API memory tool (`memory_20250818`) ≠ MCP knowledge-graph server. `Dreams`/auto-dream is a Managed-Agents-API Research Preview, **buggily wired in Claude Code** (`/dream` errors) — don't trust dreamed memory unread.

### Audit moves (run [audit-rubric.md](audit-rubric.md))
1. **Enforcement-as-prose is the #1 finding** — any "never push to main / never edit .env" with no hook/`permissions.deny` backing = BLOCKER.
2. **Size:** flag CLAUDE.md >200 lines (and >1,000 = canonical bad); count `@import` expansions in the budget.
3. **Placement:** heavy area-specific content unconditional → should be path-scoped / a skill.
4. **Consistency/freshness:** contradictions (picked arbitrarily), stale model-workarounds, deprecated model IDs.
5. **Interop:** `AGENTS.md` with no `CLAUDE.md` bridge (`@AGENTS.md`) = silent miss.
6. **Security:** committed secrets/PII (BLOCKER); `permissions.deny` for `.env*`/`*.pem`; `~/.claude/CLAUDE.md` integrity.
7. **Static vs runtime:** whether a file *loads* (`/memory`), what it *costs* (`/context`), managed in effect (`/status`), dead skills (OTEL) need a live session — mark "requires runtime verification," don't pass them statically.
8. **Rank fixes by severity × blast radius.** Existing graders to run/replicate: **agnix**, **cclint**, the 50-pt scorecard, the 100-pt audit-prompt.

---
*Full detail & sources: [audit-rubric.md](audit-rubric.md) · [claude-md-anatomy.md](claude-md-anatomy.md) · [rules-directory.md](rules-directory.md) · [choosing-the-right-mechanism.md](choosing-the-right-mechanism.md) · [context-and-token-economics.md](context-and-token-economics.md) · [adherence-and-anti-patterns.md](adherence-and-anti-patterns.md) · [large-codebase-patterns.md](large-codebase-patterns.md) · [auto-memory-and-self-improvement.md](auto-memory-and-self-improvement.md) · [sources.md](sources.md)*

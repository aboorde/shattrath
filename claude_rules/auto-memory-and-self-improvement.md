# Auto Memory & Self-Improvement

*The Claude-written sibling to CLAUDE.md: where learnings are stored, why nothing prunes them, and how to govern a write-only knowledge store. Compiled 2026-06-04; the Dreams/auto-dream section is fast-moving — re-verify before trusting.*

**Mental model.** Three unrelated things share the word "memory" in the Claude ecosystem, and an auditor's first job is to identify which one a repo is actually using. The load-bearing governance fact: as of 2026-06-04 Claude Code ships **no automatic pruning or consolidation** of its auto memory (the auto-prune request #37102 was closed *not planned*, and auto-dream is only partially/buggily wired). So a single wrong "learning" Claude saves persists in context **every future session until a human deletes it**. Auto memory is Claude's noisy, per-repo, per-machine private notebook; `CLAUDE.md`/`.claude/rules/` are the human-owned, version-controlled, team-shared source of truth. Treat auto memory as **untrusted input that loads into context every session — audit it like a dependency.**

---

## 1. The three "memory" systems (disambiguation)

Auditors routinely find the wrong one configured for the goal. They differ in **who owns storage**, **where it lives**, and **what governs it**.

| System | What it is | Storage owner / location | How invoked | Shared? | Governance reality |
|---|---|---|---|---|---|
| **Built-in auto memory** (Claude Code, **v2.1.59+**) | Claude-written learnings; the sibling to CLAUDE.md | Claude Code, at `~/.claude/projects/<project>/memory/`, indexed by `MEMORY.md` (source: https://code.claude.com/docs/en/memory) | Claude writes on "remember X"; `/memory` to browse/toggle | **No — machine-local**, never across machines/cloud (source: https://code.claude.com/docs/en/memory) | **Manual prune only** (see §4) |
| **API memory tool** (`type: memory_20250818`, `name: memory`) | A client-hosted `/memories` file directory Claude drives with `view`/`create`/`str_replace`/`insert`/`delete`/`rename` | **You** — "operates client-side: you control where and how the data is stored through your own infrastructure" (source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool) | `tools:[{type:"memory_20250818"}]` in API calls | Whatever you build | You implement path-traversal protection + storage; not Claude Code auto memory |
| **MCP knowledge-graph memory server** (`@modelcontextprotocol/server-memory`) | A "knowledge graph-based persistent memory system" of entities/relations/observations (source: https://github.com/modelcontextprotocol/servers/tree/main/src/memory) | The MCP server, default file via `MEMORY_FILE_PATH` | Connected like any MCP server; entity/relation tools | Wherever the file lives | Separate store; not auto memory, not the API tool |

**Audit tells:** `~/.claude/projects/.../memory/` + the `/memory` command = built-in auto memory; `tools:[{type:"memory_20250818"}]` in API code = API memory tool; an MCP server entry = knowledge-graph memory. Configuring the wrong one yields persistence or sharing that was never there (source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool).

Both built-in auto memory **and** CLAUDE.md load every session but are explicitly **context, not enforced configuration**: "To block an action regardless of what Claude decides, use a PreToolUse hook instead" (source: https://code.claude.com/docs/en/memory). A saved "always do X" is a suggestion, never a guarantee.

---

## 2. Mechanics of built-in auto memory

| Property | Behavior | Source |
|---|---|---|
| **Location** | `~/.claude/projects/<project>/memory/`; derived from the **git repo**, so all worktrees and subdirectories of one repo share **one** directory | https://code.claude.com/docs/en/memory |
| **Scope** | **Machine-local**; "Files are not shared across machines or cloud environments" | https://code.claude.com/docs/en/memory |
| **Index** | `MEMORY.md` "acts as an index of the memory directory… using MEMORY.md to keep track of what's stored where" | https://code.claude.com/docs/en/memory |
| **Load cap** | "The first **200 lines** of MEMORY.md, or the first **25KB**, whichever comes first, are loaded at the start of every conversation. Content beyond that threshold is **not loaded** at session start." | https://code.claude.com/docs/en/memory |
| **Topic files** | `debugging.md`, `api-conventions.md`, etc. are **not** loaded at startup; Claude reads them on demand with standard file tools | https://code.claude.com/docs/en/memory |
| **Relocation** | `autoMemoryDirectory` moves the store (absolute path or `~/` only), readable from any settings scope; in project `settings.json`/`settings.local.json` it is honored **only after the workspace trust dialog** — the same gate as hooks | https://code.claude.com/docs/en/memory |

> **Auditor note on relocation:** because `autoMemoryDirectory` is trust-gated like hooks, a repo that redirects memory into the tree is asking for the same trust grant as arbitrary hook execution. Flag any project-scoped `autoMemoryDirectory` and confirm where it points. For the full per-feature context-cost picture and the 200-line rationale, see [context-and-token-economics.md](context-and-token-economics.md).

---

## 3. Index discipline (newest notes silently don't load)

The 200-line / 25KB cap is a hard truncation at session start, so a bloated `MEMORY.md` means **Claude silently can't see its own newest notes** — the single most common reason auto memory "stops working."

- **Keep `MEMORY.md` a thin pointer index**, well under 200 lines / 25KB; push detail into on-demand topic files (source: https://code.claude.com/docs/en/memory).
- Order matters: anything past the cutoff is invisible until Claude is told to open the topic file directly.
- **Anti-pattern:** `MEMORY.md` as a single growing content dump. Everything below line 200 / 25KB is dropped each session, so the newest or most important entry can simply never load (source: https://code.claude.com/docs/en/memory).
- **Audit action:** open the auto-memory folder, count `MEMORY.md` lines/bytes, and confirm it reads as an index of pointers, not prose.

---

## 4. The manual-prune reality (a bad learning persists)

There is **no shipped auto-prune.** A request for an automatic memory-review/pruning mechanism — cross-referencing memories against skills/CLAUDE.md and flagging redundant/stale entries — was filed in the official repo and **closed as not planned** (labeled stale): "cleanup is entirely manual: the user must remember to ask Claude to review its memories and cross-reference them against other context sources" (source: https://github.com/anthropics/claude-code/issues/37102).

Consequence — the **core risk an auditor must surface:** a wrong fact Claude saves loads into context **every session until a human deletes it**, silently degrading all future work in the repo.

**Govern it like a dependency:**

- Periodically run **`/memory`** — it lists loaded files, toggles auto memory, and links the folder so you can read/edit/delete what Claude saved; "Auto memory files are plain markdown you can edit or delete at any time" (source: https://code.claude.com/docs/en/memory).
- Read **every topic file** for wrong/stale "learnings," not just `MEMORY.md`. The failure mode (one bad entry propagating across every session) is invisible from the index alone.
- Schedule this as recurring hygiene, the same cadence you'd review a pinned dependency. Pair with the live-audit commands `/memory`, `/context`, `/status`, `/doctor`.

See [adherence-and-anti-patterns.md](adherence-and-anti-patterns.md) for the write-only-knowledge warning, and [audit-rubric.md](audit-rubric.md) for where memory hygiene scores in the rubric.

---

## 5. The promotion path: auto-memory → CLAUDE.md / rules

Machine-local auto memory **cannot be the home of shared knowledge by design** — it never reaches a teammate or CI. The only way a learned convention becomes team knowledge is to **graduate it into version-controlled CLAUDE.md / `.claude/rules/`** (source: https://code.claude.com/docs/en/memory).

| Layer | Authority | Shared? | When to keep a fact here |
|---|---|---|---|
| Auto memory (`MEMORY.md` + topic files) | Claude-learned patterns | No (machine-local) | One machine's discovered build/debug quirks; transient, unstable findings |
| `CLAUDE.md` / `.claude/rules/` | Human-controlled, version-controlled | Yes | Build commands, conventions, layout, always-do-X rules; any fact that recurs and should hold every session and reach the team |

- **The promotion is user-driven, not automatic.** "When you ask Claude to remember something… Claude saves it to auto memory. To add instructions to CLAUDE.md instead, ask Claude directly, like *'add this to CLAUDE.md,'* or edit the file yourself via `/memory`" (source: https://code.claude.com/docs/en/memory).
- **Promote when** a fact has stabilized and should be shared, enforced-by-convention, and survive across machines.
- **Path-scope it** into `.claude/rules/` when it only matters for part of the codebase (see [rules-directory.md](rules-directory.md) for path-glob mechanics; note path-scoped rules are version-fragile).
- **Authority split (community framing):** "CLAUDE.md provides authoritative rules you control"; auto memory is Claude-learned. When they conflict, CLAUDE.md is the human-owned instruction layer — promote a stabilized learning out of memory to make it authoritative and shared (source: https://claudefa.st/blog/guide/mechanics/auto-dream). *Note:* the precise precedence between a contradicting auto-memory line and a CLAUDE.md line is **not** an official rule — both are "context, not enforced," so a conflict may be resolved arbitrarily. Don't rely on "CLAUDE.md always wins"; remove the contradiction.

For the full three-axis routing rubric (load-timing / context-cost / guarantee-vs-guidance) and the promotion ladder, see [choosing-the-right-mechanism.md](choosing-the-right-mechanism.md).

---

## 6. Subagent memory: the one auto-written memory you *can* share

Subagents keep their own persistent memory via a **`memory:` frontmatter field** with three scopes (source: https://code.claude.com/docs/en/sub-agents):

| `memory:` scope | Directory | Git-trackable / shared? |
|---|---|---|
| `user` | `~/.claude/agent-memory/<name>/` | No (per-machine) |
| **`project`** (recommended) | `.claude/agent-memory/<name>/` | **Yes — version-controllable / shareable** |
| `local` | `.claude/agent-memory-local/<name>/` | No (gitignored) |

- Enabling `memory:` auto-injects memory-read/write instructions, auto-enables `Read`/`Write`/`Edit`, and includes "the first 200 lines or 25KB of MEMORY.md in the memory directory, whichever comes first, with instructions to curate MEMORY.md if it exceeds that limit" (source: https://code.claude.com/docs/en/sub-agents) — the same cap as main-agent memory.
- **`project` is the recommended default scope** because "It makes subagent knowledge shareable via version control" (source: https://code.claude.com/docs/en/sub-agents). This is the **inverse** of main-agent auto memory (always machine-local) — and the **one auto-written memory you can git-track, review in PRs, and share with a team.**
- Make accumulation reliable: put explicit *"consult your memory before / save what you learned after"* instructions in the subagent file. This is the only first-class path to a shareable, reviewable, self-improving agent knowledge base in Claude Code.

---

## 7. Dreams / auto-dream — STATUS (fast-moving, do **not** treat as stable)

"Dreams" is **memory consolidation** — merge duplicates, replace stale/contradicted entries, surface insights. Critically, it is an **official feature of the Managed Agents API**, **not** stable Claude Code.

**Official (Managed Agents API, Research Preview):** requires beta headers `managed-agents-2026-04-01` and `dreaming-2026-04-21`; it is **non-destructive** — "The input store is never modified, so you can review the output and discard it if you don't like the result" (source: https://platform.claude.com/docs/en/managed-agents/dreams). It emits a **separate output store** you attach only after review; an `instructions` field steers curation (e.g. "focus on coding-style preferences; ignore one-off debugging notes"). **This review-then-adopt pattern is the officially sanctioned governance model** — emulate it even when consolidating manually.

**In Claude Code specifically (2026-06, partially/buggily wired):**

| Symptom | Detail | Source |
|---|---|---|
| `/dream` errors | An `autoDreamEnabled` setting and a `/memory` toggle exist, but `/dream` returns **"Unknown skill: dream"** and no background task starts — the handler/spawner aren't connected | https://github.com/anthropics/claude-code/issues/38461 |
| Output unauditable | Dreamed memories can be "inaccurately named, factually unverified… and impossible to audit" — only a "last ran Xs ago" indicator, no log of what a run created/modified/removed | https://github.com/anthropics/claude-code/issues/38493 |

**Do not assume Claude Code auto-consolidates memory** (as of 2026-06-04), and **never trust a dreamed memory without human review** (source: https://github.com/anthropics/claude-code/issues/38493). Community write-ups describing exact triggers (24h + 5 sessions, four phases, <200-line trim, lock file) are **speculation, not spec** — see Confidence & gaps.

---

## 8. Security — `~/.claude/projects` is world-readable plaintext

Auto memory and the session transcripts under `~/.claude/projects/` are **world-readable plaintext (`-rw-r--r--`)** with "no real application-layer access control," no audit trail, and no retention policy; `.jsonl` transcripts can capture secrets — bearer tokens, `API_KEY`/`SECRET`/`PASSWORD` on command lines (community report, source: https://serendb.com/blog/claude-code-local-memory-security-risk).

**Audit actions (don't skip these as "just local notes"):**

```bash
# grep memory + transcript store for leaked secrets
rg -nS '(API_KEY|SECRET|PASSWORD|PRIVATE_KEY|TOKEN|Authorization: Bearer)' ~/.claude/projects
# enumerate auto-memory topic files for review
find ~/.claude/projects -path '*/memory/*.md'
```

Then tighten permissions and periodically clear old transcripts/memory. This is a concrete, often-missed attack surface (source: https://serendb.com/blog/claude-code-local-memory-security-risk).

---

## 9. Self-improving loops, done safely

Because auto-dream is unwired/unauditable (§7), build self-improvement on **shipped, deterministic, auditable** primitives — and **always pair writing with pruning** (a write-only knowledge store is the anti-pattern; see [adherence-and-anti-patterns.md](adherence-and-anti-patterns.md)).

- **`Stop` hook** to propose CLAUDE.md updates at end of session (source: https://code.claude.com/docs/en/memory).
- **`InstructionsLoaded` hook** logs exactly which instruction/memory files loaded — useful to catch a `MEMORY.md` that quietly stopped loading (e.g. when it grew past the 200-line cap) (source: https://code.claude.com/docs/en/memory).
- **Recurring manual hygiene prompt:** "review my MEMORY.md against CLAUDE.md/skills and flag redundant or contradicted entries." This is the human substitute for the not-planned auto-prune (#37102).
- **If you adopt the Dreams API**, exploit its non-destructive design: review the separate output store, then attach or discard (source: https://platform.claude.com/docs/en/managed-agents/dreams).

**Rule:** every self-improving loop must include a pruning step. An accumulate-only loop guarantees that wrong, stale, and contradicted entries compound across every future session, which is exactly the failure §4 describes.

---

## Sources

1. https://code.claude.com/docs/en/memory — *(official)* Auto memory: location, machine-local scope, 200-line/25KB cap, MEMORY.md index, manual prune, `/memory`, `autoMemoryDirectory`, promotion, context-not-config, Stop/InstructionsLoaded hooks.
2. https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool — *(official)* API memory tool (`memory_20250818`), client-hosted `/memories`, path-traversal responsibility.
3. https://github.com/modelcontextprotocol/servers/tree/main/src/memory — *(official)* MCP knowledge-graph memory server (`@modelcontextprotocol/server-memory`).
4. https://github.com/anthropics/claude-code/issues/37102 — *(official issue)* Auto-prune request closed *not planned*; cleanup is manual.
5. https://code.claude.com/docs/en/sub-agents — *(official)* Subagent `memory:` frontmatter (user/project/local); `project` recommended/shareable; 200-line/25KB injection.
6. https://platform.claude.com/docs/en/managed-agents/dreams — *(official)* Dreams Managed Agents API Research Preview; beta headers; non-destructive separate output store; `instructions` field.
7. https://github.com/anthropics/claude-code/issues/38461 — *(official issue)* `/dream` returns "Unknown skill"; auto-dream not wired in Claude Code.
8. https://github.com/anthropics/claude-code/issues/38493 — *(official issue)* Dreamed-memory output inaccurate/unverified/unauditable.
9. https://serendb.com/blog/claude-code-local-memory-security-risk — *(community)* `~/.claude/projects` world-readable plaintext; transcripts capture secrets; grep guidance.
10. https://claudefa.st/blog/guide/mechanics/auto-dream — *(community)* CLAUDE.md-as-authoritative authority-split framing.

## Confidence & gaps

- **Verified official (safe to state):** storage location, machine-local-per-repo scope, the 200-line/25KB load cap, MEMORY.md-as-index, on-demand topic files, manual prune, `/memory`, `autoMemoryDirectory` (trust-gated), the user-driven promotion path, "context not enforced configuration," subagent `memory:` scopes with `project` recommended, and the Dreams API's non-destructive design — all from `code.claude.com`/`platform.claude.com` (sources 1, 2, 5, 6). The three official GitHub issues (4, 7, 8) are official-repo signals, not docs — treat #37102's *not-planned* status and the auto-dream bugs as current but version-sensitive.
- **Community / inference (labeled in-line):** the world-readable-plaintext security claim and grep recipe (source 9, community); the "CLAUDE.md is authoritative when it conflicts with auto memory" framing (source 10, community) — reasonable but **not** an official precedence rule, since both layers are merely "context, not enforced."
- **Fast-moving — re-verify:** **Dreams/auto-dream in Claude Code** is in flux. Officially a Managed Agents API Research Preview (beta header `dreaming-2026-04-21`) but only partially/buggily wired in Claude Code as of 2026-06 (`autoDreamEnabled` + `/memory` toggle exist; `/dream` errors; output unauditable). Exact Claude Code rollout status/version is **unconfirmed and likely to change** — do not document it as a stable Claude Code feature without re-checking #38461 / #38493 and the Dreams docs.
- **Community speculation excluded:** detailed auto-dream trigger mechanics (24h + 5 sessions, four phases, lock file, read-only-to-memory) from non-official write-ups are **not** in any official page and may describe unreleased/changing internal behavior — omitted as spec. One community article framing auto memory as living *inside* CLAUDE.md (no separate `MEMORY.md`) contradicts current official docs (separate store, v2.1.59+) and appears stale/pre-feature — excluded.
- **Open inference flagged:** whether **main-agent** auto memory should ever be git-tracked is not directly addressed officially; docs say machine-local (implied: no), though `autoMemoryDirectory` could relocate it into a repo. Only **subagent `memory:project`** is the explicitly sanctioned git-trackable path.
- **Version sensitivity:** auto memory is **v2.1.59+**; the 200-line/25KB cap and `memory:` frontmatter are current-as-of 2026-06-04 and could change with releases.

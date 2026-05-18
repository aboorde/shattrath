# Memory in the Claude Ecosystem — A Disambiguation Guide for Claude Cowork Users

> Scope: this document separates the **four distinct "memory" mechanisms** Anthropic ships, explains exactly which one **Claude Cowork** uses and how a power user curates it, and extracts sourced best practices. The terms overlap in marketing but are different systems with different storage, scope, and controls. Verified against official Anthropic docs/help-center/blog as of **2026-05-18**.

---

## 0. TL;DR

- **Claude Cowork uses file-based project memory** (a human-readable `memory.md`-style markdown file stored **locally on the user's machine**, inside a Cowork **Project**). It is the *same lineage* as Claude Code's auto-memory, not the cloud "memory synthesis" used by Claude.ai chat. (sources 8, 9, 11, 12)
- **Memory in Cowork is scoped to a Project. It does NOT persist across standalone (folder-only) Cowork sessions, and it is NOT team-shared** — project data including memory is stored locally per user. (sources 8, 9)
- The cloud feature most people call "Claude's memory" (remembering you across chats, per-project memory summaries, Incognito) is a **separate** product feature that lives in **Claude.ai chat / projects**, not the same thing as Cowork's file memory. (sources 1, 2, 3)
- For repeatable workflows, the right curation stack is: **Skills** (procedures, load on demand) + **Project instructions** (always-on rules/tone) + **Project memory** (learned preferences/state) + **Project context/connectors** (the data). Don't collapse these into one. (sources 8, 13, 15)

---

## 1. The four memory mechanisms (disambiguation)

There are four. They are easy to confuse because Anthropic reuses the words "memory" and "projects" across products.

### 1.1 Claude.ai consumer/work chat memory ("memory synthesis" + chat search)

This is the product feature where Claude "remembers" you across conversations.

- **What it is:** Claude automatically synthesizes your conversation history into a knowledge base. The system maintains **separate memory spaces: one for general chats outside projects, and a dedicated memory for each individual project**. (source 1) It learns "your role, projects, and professional context," "communication preferences and working style," "technical preferences and coding style," and "project details and ongoing work." (source 1)
- **Synthesis cadence:** updates run roughly **every 24 hours** as conversations are created, modified, or deleted; deleted conversations are removed from memory synthesis. (source 1)
- **Two tools under the hood:** memory/search surface as function tools — `conversation_search` and `recent_chats` — so you can see when Claude pulls prior context; retrieval is RAG-style. (sources 1, 4)
- **Enable / pause / reset:** **Settings > Capabilities**. "Pause memory" keeps existing memories but neither uses nor creates them; "Reset memory" *permanently deletes all memories including project memories* and cannot be undone. (source 1)
- **View / edit:** Settings > Capabilities > "View and edit memory" opens a **Manage memory** modal of everything Claude remembers; you can also tell Claude in-chat what to remember/ignore and it adjusts the referenced memories. (sources 1, 3)
- **Incognito chats:** temporary chats not saved to history; Claude won't pull from them when searching past conversations. Available to all users. (sources 1, 3)
- **Plans:** memory is available across Free/Pro/Max/Team/Enterprise; **chat search of past conversations requires a paid plan** (Pro, Max, Team, Enterprise). (source 1)
- **Project memory (Claude.ai):** "If you use projects, Claude creates a separate memory for each project," keeping e.g. product-launch planning separate from unrelated work. (source 3)
- **Enterprise/Team:** org-wide memory defaults to enabled; Enterprise **Owner/Primary Owner** can disable at **Organization settings > Capabilities**, which **immediately and permanently deletes all memory synthesis data for all users**. Team plans **do not** have org-level memory controls (user-level only). (sources 1, 7)
- **Rollout dates:** announced **2025-09-11**, initially Team/Enterprise; Pro and Max received it **2025-10-23**. (source 2)

> Note: This is the mechanism the "Bringing memory to teams" blog describes (source 2). It is **cloud-side synthesis**, distinct from the file-based memory below.

### 1.2 The Agent SDK / API memory tool (file-backed, client-executed)

This is a developer primitive, not an end-user feature.

- **What it is:** a client-side tool letting Claude `create / read / update / delete` files in a `/memories` directory that persist between sessions, so agents build knowledge over time without keeping everything in context. (source 5)
- **Tool identifier / beta header:** tool `type` is **`memory_20250818`**; context management ships under beta header **`context-management-2025-06-27`**. (sources 5, 6)
- **Client-executed:** Claude emits tool calls; **your application performs the file operations** wherever you choose (filesystem, DB, encrypted store). SDK helpers: `BetaAbstractMemoryTool` (Python), `betaMemoryTool` (TypeScript). (source 5)
- **Commands:** `view`, `create`, `str_replace`, `insert`, `delete`, `rename`, each with prescribed return/error strings. (source 5)
- **Trigger behavior:** when enabled, an instruction is auto-injected: *"IMPORTANT: ALWAYS VIEW YOUR MEMORY DIRECTORY BEFORE DOING ANYTHING ELSE … ASSUME INTERRUPTION: Your context window might be reset at any moment."* So Claude `view`s `/memories` before work and records progress as it goes. (source 5)
- **Pairs with context editing & compaction:** memory persists important info **across compaction/context-editing boundaries** so nothing critical is lost when older tool results are cleared/summarized. (sources 5, 6)
- **Security guidance (built into the docs):** mandatory **path-traversal protection** (validate paths start with `/memories`, canonicalize, reject `../`/`..\\`/`%2e%2e%2f`); Claude *usually* refuses to write sensitive info but you should strip it; cap file sizes/paginate; expire stale files. (source 5)
- **Privacy:** memory tool **is eligible for Zero Data Retention**; with a ZDR arrangement, data isn't stored after the response. (source 5)
- **Context engineering basis:** this is Anthropic's "just-in-time context retrieval" / structured note-taking pattern (memory tool released in public beta, blog dated **2025-09-29**). (source 10)

### 1.3 Claude Code memory (CLAUDE.md hierarchy + auto memory) — for contrast

Claude Code has **two complementary** systems, both loaded at the start of every session, treated as context (not enforced config). (source 11)

- **CLAUDE.md files (you write):** loaded by walking up the directory tree; scopes in load order — **managed policy** (org, cannot be excluded) → **user** (`~/.claude/CLAUDE.md`) → **project** (`./CLAUDE.md` or `./.claude/CLAUDE.md`) → **local** (`./CLAUDE.local.md`, gitignored). Supports `@path` imports (max depth 5), `.claude/rules/` with optional path-scoped frontmatter. Target <200 lines; loaded *in full* regardless of length. (source 11)
- **Auto memory (Claude writes):** Claude saves its own learnings (build commands, debugging insights, preferences) to `~/.claude/projects/<project>/memory/` with a `MEMORY.md` index plus topic files. **Only the first 200 lines / 25 KB of `MEMORY.md` load at session start**; topic files load on demand. Per-repo, **machine-local**, shared across worktrees, not synced across machines. On by default (requires Claude Code v2.1.59+); toggle via `/memory`, `autoMemoryEnabled`, or `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`. Audit/edit via `/memory`. (source 11)

> This is the architectural ancestor of Cowork's memory. Note Cowork Projects are *not* yet in Claude Code: "Projects are only available in Cowork, not in Claude Code. Support for Claude Code is planned." (source 8)

### 1.4 Claude Cowork project memory (the one Cowork actually uses)

See Section 2 — this is the focus.

---

## 2. What memory does Claude Cowork actually use?

**Claude Cowork is the agentic architecture that powers Claude Code, brought into Claude Desktop without the terminal.** (source 12) Because it shares Claude Code's lineage, its memory is **file-based and local**, not the Claude.ai cloud synthesis of §1.1.

### 2.1 The mechanism

- **Memory is enabled for Cowork *Projects*.** Per the official help center: *"Memory is enabled for Cowork projects. This means Claude can remember context from tasks you've run in a project and apply it to future tasks in the same project."* (source 8)
- **Standalone sessions get no memory.** *"Memory is supported within projects but is not retained across standalone Cowork sessions."* A folder picked without a Project starts fresh each time and gets **no memory, no project instructions, no scheduled tasks**. (sources 9, 12)
- **Storage = local markdown file.** Anthropic's help center confirms project data including memory is stored **locally on each user's computer** (source 8). Community deep-dives (clearly marked as non-official) consistently report the concrete shape: Claude writes preferences/learnings to a **`memory.md`** (and a `claude.md`-style instructions file) inside the project folder, viewable via a **memory icon in the top-left of the project view** or by opening the markdown file directly. (sources 14, 15 — community)
  - Anthropic's own docs **do not publish the exact filename/UI affordance** for Cowork memory; treat the `memory.md` filename and memory-icon details as community-reported until Anthropic documents them. The *behavior* (project-scoped, local, markdown, user-editable, "tell Claude to remember") is officially confirmed (source 8); the *exact file/UI* is inferred.

### 2.2 A Cowork Project's four distinct parts (don't conflate)

Official help center lists them explicitly (source 8):

| Part | What it is | Lifetime |
|---|---|---|
| **Instructions** | Tone, formatting, rules guiding *every* task in the project (always applied) | Static until you edit |
| **Scheduled tasks** | Recurring tasks specific to the project (`/schedule`, or "Scheduled" sidebar; run only while the computer is awake and Desktop is open) | Until deleted |
| **Context** | A local folder, a linked chat project, or pasted URLs Claude references | Static until you change it |
| **Memory** | What Claude *learns* from running tasks in the project, applied to future tasks | Accumulates; user-curated |

### 2.3 Scoping summary for Cowork memory

- **Session (standalone, folder-only):** ❌ no persistent memory. (source 9)
- **Project:** ✅ memory persists across all tasks in that project; **siloed** — what Claude learns in "Sales Reporting" does not bleed into another project or a folder session. (sources 8, 14)
- **Account/user:** memory + project data are **per-user, stored locally on that user's machine**. (source 8)
- **Team/Enterprise:** **not team-shared and not centrally managed.** *"Project data (tasks and memory) is stored locally on each user's computer … This data is not subject to Anthropic's standard data retention policies … cannot be centrally managed or exported by admins."* There is no separate admin toggle for projects/memory; admins control Cowork at the **org on/off** level (Organization settings > Capabilities) and via **plugin/MCP marketplace** + **network egress** settings. (source 9)

### 2.4 How memory interacts with Skills and connectors in Cowork

- **Skills ≠ memory.** Skills are modular capabilities (a `SKILL.md` + optional scripts/resources) loaded by **progressive disclosure**: Level 1 metadata (~100 tokens) always in the system prompt, Level 2 SKILL.md body (<5k tokens) loaded **only when triggered**, Level 3 resources/scripts loaded on demand. Cowork ships built-in document skills (**pptx, xlsx, docx, pdf**, plus canvas-design per community reports) and supports custom Skills. (sources 13, 9-community-context) Skills carry *how to do a task*; memory carries *what Claude learned about your work and preferences*. They compose: a Skill defines the procedure, memory remembers your tweaks to it.
- **Skills sharing scope is per-user on claude.ai-style surfaces:** "Custom Skills are individual to each user; not shared organization-wide and cannot be centrally managed by admins." (source 13) Same per-user pattern as Cowork memory.
- **Connectors/MCP** (Gmail, Google Drive, Notion, Calendar, Figma, etc.) are the *data reach*, governed by network-egress + plugin/marketplace admin settings; network settings apply **only to new Cowork sessions**, not retroactively. (source 9) Connectors are not memory — Claude reads live data through them, then may record learned preferences (not the raw data) into project memory.

### 2.5 Cowork memory vs. Claude.ai chat memory (critical distinction)

| | Cowork project memory | Claude.ai chat memory (§1.1) |
|---|---|---|
| Storage | Local markdown file on user's machine (source 8) | Cloud "memory synthesis" (source 1) |
| Scope unit | Cowork **Project** | Account-wide *or* per Claude.ai chat **Project** (source 3) |
| Standalone session | No memory (source 9) | N/A (chat always has account memory unless paused/incognito) |
| Admin org control | Only org on/off for Cowork; memory not separately managed (source 9) | Enterprise Owner can disable org-wide, deletes all synthesis (source 1) |
| User curation | Edit the markdown / "tell Claude to remember" (source 8, 14) | Manage-memory modal, pause/reset, in-chat instructions (source 1) |

---

## 3. Comparison table — all four mechanisms

| Dimension | Claude.ai chat memory (§1.1) | API memory tool (§1.2) | Claude Code memory (§1.3) | **Cowork project memory (§1.4)** |
|---|---|---|---|---|
| Audience | Claude.ai end users | Developers (Agent SDK/API) | Claude Code users | **Cowork users (power users)** |
| Storage | Cloud synthesis (Anthropic-side) | Client-side, dev-chosen (`/memories`) | Local files (`CLAUDE.md`, `~/.claude/projects/.../memory/`) | **Local markdown in the Project folder** |
| Who writes it | Claude (auto) + user edits | Claude via tool calls; app executes | User (CLAUDE.md) + Claude (auto memory) | **Claude (learned) + user ("remember this") + manual edit** |
| Scope | Account, or per Claude.ai project | Per dev-defined store/session | Per repo (auto memory); hierarchical (CLAUDE.md) | **Per Cowork Project; per-user; local** |
| Persists across sessions | Yes (synthesis) | Yes (files persist) | Yes (files persist) | **Yes within a Project; NO in standalone sessions** |
| Trigger | RAG via `conversation_search`/`recent_chats` | Auto `view /memories` before tasks | Loaded at session start (`MEMORY.md` head) | **Auto-referenced for tasks in the Project** |
| User view/edit | Settings > Capabilities modal; in-chat | App-dependent (it's your store) | `/memory` command; edit markdown | **Memory icon in project view / edit markdown** (UI detail community-reported) |
| Off switch | Pause/Reset; Incognito; Enterprise org disable | Don't include the tool | `autoMemoryEnabled:false` / env var | **Don't use a Project (use a folder session); org-disable Cowork** |
| Identifier / dates | Announced 2025-09-11; Pro/Max 2025-10-23 | `memory_20250818`; `context-management-2025-06-27` | Auto memory needs Claude Code v2.1.59+ | Cowork Projects (community: ~March 2026) |
| ZDR / training | Follows chat retention; commercial data not trained by default | **ZDR-eligible**; not stored under ZDR | Local files (not Anthropic-stored) | Local; *not subject to standard retention* (source 9) |

---

## 4. Best practices (sourced)

### 4.1 Memory hygiene

- **Keep memory atomic, coherent, current.** Anthropic's own anti-clutter instruction: *"always try to keep its content up-to-date, coherent and organized. You can rename or delete files that are no longer relevant. Do not create new files unless necessary."* (source 5). The Claude Code guidance reinforces: review periodically and remove **outdated or conflicting** entries — *"if two rules contradict, Claude may pick one arbitrarily."* (source 11)
- **Scope what gets written.** You can constrain memory: *"Only write down information relevant to <topic> in your memory system."* (source 5) For a power user, tell Cowork to remember *workflow preferences, project naming conventions, recurring-report formats* — not transient task data.
- **Don't store secrets.** Claude *usually* refuses to write sensitive info, but the docs explicitly recommend external stripping/validation — don't rely on the refusal. (source 5) Treat the local `memory.md` as readable plaintext; never paste tokens/passwords expecting memory to protect them.
- **Prune on a cadence.** Auto memory keeps only the first 200 lines / 25 KB of the index loaded (source 11); long, stale memory is both ignored *and* a poisoning risk. Periodically open the Cowork memory file (or `/memory` in Code) and delete dead entries. (sources 5, 11)
- **One feature/decision at a time; verify before recording.** Anthropic's multi-session pattern: only mark something complete *after end-to-end verification*, keeping the progress log trustworthy and preventing scope creep across sessions. (source 5)

### 4.2 Memory vs. project knowledge vs. skills vs. instructions — which to use

| Need | Use | Why (sourced) |
|---|---|---|
| A repeatable **procedure/workflow** (e.g., "generate the review deck this way") | **Skill** | Loads on demand via progressive disclosure; 100 skills cost ~0 context until triggered; portable/versioned (source 13) |
| **Always-on rules/tone/format** for a project | **Project instructions** (Cowork) / **CLAUDE.md** (Code) | Applied to *every* task/session; not on-demand (sources 8, 11) |
| **What Claude learned** about your preferences/state over time | **Memory** | Accumulates from corrections without manual authoring (sources 5, 11) |
| The **data/source** to act on | **Project context / connectors** | Folder, linked project, URL, Gmail/Drive/Notion (sources 8, 9) |
| One-off guidance | **A prompt** | Skills exist precisely to avoid repeating prompts (source 13) |

Rule of thumb (from Claude Code docs, transferable): if an entry is a **multi-step procedure or only matters sometimes → Skill or path-scoped rule**, not memory/instructions. Memory is for *facts/preferences Claude should hold*, instructions for *"always do X."* (source 11)

### 4.3 Governance, privacy, training

- **Commercial data is not used for training by default.** Claude for Work/Enterprise/API inputs/outputs are not trained on unless you explicitly submit feedback/bugs; feedback may be retained up to 5 years and is de-linked from user/account IDs before use. (sources, Privacy Center 16; source 7)
- **Cowork project data is local and outside standard retention** — and therefore **cannot be exported/managed/audited by admins.** Plan governance around this: there is no central memory console for Cowork. (source 9)
- **Claude.ai chat memory** follows chat retention; Enterprise Owners can hard-disable org-wide (irreversibly deleting all synthesis); Team plans lack org controls. (sources 1, 7)
- **Enterprise admin levers for Cowork:** org on/off (Organization settings > Capabilities), Enterprise role/group-based enablement, plugin/MCP marketplace policy (auto-installed/available/required/hidden), and network-egress (applies to new sessions only). No granular per-user/role controls on Team plans. (source 9)
- **ZDR:** the API memory tool and context editing are ZDR-eligible; **Skills are NOT ZDR-eligible** (retained under standard policy). Relevant if a power user's workflow blends skills with sensitive workstreams. (sources 5, 13)

### 4.4 Failure modes & mitigations

- **Memory bloat** → degraded focus + index truncation (only first 200 lines/25 KB loaded). *Mitigation:* keep the index concise, push detail to topic files, prune regularly. (sources 5, 11)
- **Stale/contradictory memory poisoning behavior** → Claude picks a rule arbitrarily. *Mitigation:* periodic review for conflicts; delete superseded entries; prefer specific, verifiable statements. (source 11)
- **Cross-project / cross-context leakage** → mitigated by design: Claude.ai and Cowork both **silo memory per project**; confidential work stays out of unrelated work. *Action:* deliberately use **separate Projects** per project/workstream; don't run sensitive work in a shared/standalone session. (sources 3, 8, 14)
- **Prompt injection into memory** → a malicious file/connector/Skill could write instructions Claude later trusts. Anthropic's mitigations: mandatory path-traversal protection and sensitive-data stripping for the memory tool (source 5); **only use Skills from trusted sources** — audit SKILL.md/scripts, beware Skills fetching external URLs (data exfiltration risk) (source 13). *Action:* treat connectors/Skills like installed software; review before trusting; don't let untrusted content flow into a project whose memory then steers future work.
- **"Why did Claude forget?"** (common support ticket) → almost always because work ran in a **standalone folder session, not a Project** (no memory there), or memory was paused/incognito on the Claude.ai side. *Mitigation:* always create/attach a **Project** for recurring work. (sources 8, 9)

---

## Sources

1. https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context — Official help center: Claude.ai chat memory + search; enable/pause/reset, view-edit modal, incognito, project-scoped memory, 24h synthesis, plan availability, Enterprise org disable. (no explicit date shown; current as fetched 2026-05-18)
2. https://claude.com/blog/memory — Official Anthropic blog "Bringing memory to Claude/teams"; project-vs-account memory, Team/Enterprise rollout, Incognito, admin disable. Published **2025-09-11** (Pro/Max **2025-10-23**).
3. (within source 2/1) Project-scoped Claude.ai memory separation ("separate memory for each project"). 2025-09-11.
4. Simon Willison, https://simonwillison.net/2025/Sep/12/claude-memory/ — community deep-dive corroborating `conversation_search`/`recent_chats` tool surfacing. Community, dated 2025-09-12.
5. https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool — Official API docs: memory tool, `memory_20250818`, `/memories`, commands, auto-injected protocol, security/path-traversal, ZDR eligibility, compaction/context-editing pairing. Current as fetched 2026-05-18.
6. https://platform.claude.com/docs/en/build-with-claude/context-editing — Official: `clear_tool_uses_20250919`, `clear_thinking_20251015`, beta header `context-management-2025-06-27`, server-side behavior, memory pairing. Current as fetched.
7. https://privacy.claude.com/en/articles/7996868-is-my-data-used-for-model-training — Official Privacy Center: commercial data not trained by default, feedback retention ~5y, de-linking, admin opt-out path. Current as fetched.
8. https://support.claude.com/en/articles/14116274-organize-your-tasks-with-projects-in-claude-cowork — Official help center: Cowork Projects; "Memory is enabled for Cowork projects," project-scoped, the 4 project parts, "Projects only in Cowork, not Claude Code." Current as fetched.
9. https://support.claude.com/en/articles/13455879-use-claude-cowork-on-team-and-enterprise-plans — Official: project/task/memory stored locally per user, not centrally managed/exported, org enable/disable, plugin/MCP + network-egress governance, no granular Team controls. Marked "updated this week" (≈May 2026).
10. https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents — Official engineering blog: just-in-time retrieval, structured note-taking, memory tool public beta. Published **2025-09-29**.
11. https://code.claude.com/docs/en/memory — Official Claude Code docs: CLAUDE.md hierarchy, imports, auto memory (`~/.claude/projects/<project>/memory/`, 200-line/25 KB load), `/memory`, v2.1.59+, conflict guidance. Current as fetched.
12. https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — Official: Cowork = Claude Code architecture in Desktop; standalone vs project; "memory not retained across standalone sessions." Current as fetched.
13. https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview — Official: Agent Skills, SKILL.md, 3-level progressive disclosure, per-user (claude.ai) sharing, not ZDR-eligible, security guidance. Current as fetched.
14. https://ryanandmattdatascience.com/claude-cowork-projects/ and https://www.goodaiguide.com/guides/project-memory-claude/ — Community guides: concrete `memory.md` filename, memory-icon UI, project siloing examples. **Community/unofficial**, ~March–May 2026.
15. https://karozieminski.substack.com/p/claude-cowork-guide-plugins-memory-sub-agents-tips — Community power-user guide: Cowork writes preferences to claude.md/memory.md, skills behavior. **Community/unofficial**, 2026.
16. https://privacy.claude.com (commercial-vs-consumer training section, via source 7 fetch) — Official Privacy Center. Current as fetched.

## Confidence & gaps

- **Official / high confidence:** §1.1 Claude.ai memory mechanics, dates, Enterprise controls (sources 1, 2, 7); §1.2 API memory tool internals, identifiers, security, ZDR (source 5); §1.3 Claude Code memory model (source 11); §1.4 *behavioral* facts — Cowork memory is project-scoped, local-per-user, not team-shared, absent in standalone sessions, no central admin export (sources 8, 9, 12); Skills mechanics and sharing scope (source 13).
- **Inferred / community-sourced (flagged in text):** the **exact Cowork memory filename (`memory.md`/`claude.md`) and the "memory icon" UI affordance** are *not* published in Anthropic's official Cowork help articles I fetched — these come from reputable community guides (sources 14, 15) and from Cowork's documented Claude-Code lineage. The *behavior* is official; the *file/UI specifics* should be verified in-product before asserting them as fact.
- **Undocumented for Cowork specifically:** Anthropic does not publish (a) whether Cowork memory uses the same `MEMORY.md` index/topic-file split and 200-line/25 KB load limit as Claude Code auto memory (plausible given shared architecture, but unconfirmed), (b) any in-product editor/viewer spec for Cowork memory, (c) whether/when Cowork Projects/memory will sync to Claude Code (docs say "planned," no date — source 8).
- **Staleness risks:** Cowork shipped recently and its help articles are actively revised (source 9 marked "updated this week"). Memory feature dates (2025-09-11 / 2025-10-23) and identifiers (`memory_20250818`, `context-management-2025-06-27`) are stable as of 2026-05-18 but the beta status of the API memory tool / context editing could change. Re-verify Cowork specifics directly in the help center before stakeholder-facing commitments.

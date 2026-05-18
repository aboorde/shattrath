# Claude Cowork vs. Claude Code: A Power-User's Translation Guide

*Prepared for a technically literate power user adopting Claude Cowork, whose mental model is anchored in heavy Claude Code use. Research conducted 2026-05-18. Every non-obvious claim is sourced inline; see `## Sources` and `## Confidence & gaps` at the end. Official Anthropic sources are marked **[OFFICIAL]**; everything else is community/press and marked accordingly.*

---

## 1. Executive summary (the one-paragraph translation)

Claude Cowork and Claude Code are **two front-ends over the same agentic engine** (the harness that Anthropic generalized out of Claude Code and now ships as the Claude Agent SDK). Claude Code is the **terminal/IDE product for engineers**: full local shell, your real filesystem with the launching user's permissions, Git, plugins, hooks, `settings.json`, ~70 built-in slash commands. Claude Cowork is the **Claude Desktop product for non-engineer knowledge workers**: the same multi-step autonomy, but code/commands run inside an **isolated VM**, file access is **scoped to folders you explicitly connect**, and capability is surfaced through a **GUI plus Skills/Plugins/Connectors** rather than a terminal full of slash commands. A Claude Code power user's intuition transfers almost 1:1 at the *behavioral* level (planning, subagents, skills, MCP, memory, permissions). What changes is the **packaging**: Cowork hides the CLI machinery, so the dozens of operator slash commands they rely on (`/init`, `/review`, `/security-review`, `/hooks`, `/mcp`, `/plan`, `/loop`, `/schedule`, plugin commands) either don't exist as typed commands or are re-expressed as GUI controls and `/`-invoked **Skills**.

---

## 2. What is Claude Cowork?

### 2.1 Positioning and the problem it solves

Anthropic positions Cowork as **"Claude Code for the rest of your work"** — the agentic capability of Claude Code, but for knowledge work beyond coding (source: https://claude.com/blog/cowork-research-preview — official Anthropic blog, "Cowork: Claude Code power for knowledge work"; and search-surfaced framing of the Jan 12 2026 launch). The official help center states it directly: **"Claude Cowork brings Claude Code's agentic capabilities to Claude Desktop for knowledge work beyond coding"** (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]**).

The core pitch is **outcome over prompts**: *"Most AI assistants require users to break work into individual prompts. Claude Cowork takes the outcome and handles the rest"* and *"Delegate to Claude, delight in the result. Hand off a task, get a polished deliverable"* (sources: https://www.anthropic.com/product/claude-cowork and https://claude.com/blog/cowork-research-preview — **[OFFICIAL]**). The mental shorthand widely used (community): *Claude.ai is for thinking with Claude, Cowork is for delegating to Claude, Claude Code is for building with Claude* (source: https://mattgeer.com/difference-between-claude-ai-cowork-and-code/ — community).

### 2.2 Who it's for

Anthropic's stated audience: **"Researchers, analysts, operations teams, legal professionals, finance teams: people who work with documents, data, and files every day"** (source: https://www.anthropic.com/product/claude-cowork — **[OFFICIAL]**). Explicitly the non-developer "knowledge worker" who needs full-task agency but does not operate a command line (source: https://techcrunch.com/2026/01/12/anthropics-new-cowork-tool-offers-claude-code-without-the-code/ — community/press, TechCrunch, 2026-01-12).

### 2.3 How it relates to Claude.ai, Claude Code, and the Agent SDK

- **Shared engine.** Anthropic's engineering write-up states *"the agent harness that powers Claude Code (the Claude Code SDK) can power many other types of agents, too,"* and that this harness was renamed the **Claude Agent SDK** (source: https://claude.com/blog/building-agents-with-the-claude-agent-sdk — **[OFFICIAL]**, originally published 2025-09-29). Press at launch reported Cowork is *"built on the Claude Agent SDK using the same underlying model as Claude Code"* and is "the result similar to a sandboxed instance of Claude Code" (source: https://techcrunch.com/2026/01/12/anthropics-new-cowork-tool-offers-claude-code-without-the-code/ — community/press).
- **Relative to Claude.ai chat.** Cowork is a distinct surface *inside the Claude Desktop app* (a "Cowork tab"), separate from normal chat; chat answers questions, Cowork executes multi-step tasks against your files (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]**).
- **Relative to Claude Code.** Same agentic behavior; different environment, audience, and safety envelope (sandboxed VM vs. your real shell). They are sibling products, not the same app.
- **Billing relationship (community, but consistent):** Cowork, Claude Code in terminal/IDE, and Claude chat draw on **subscription usage limits**, whereas the Agent SDK / `claude -p` / GitHub Actions draw on a separate Agent SDK credit pool (source: https://help.apiyi.com/en/anthropic-claude-subscription-agent-sdk-billing-split-june-2026-en.html — community, treat as indicative).

### 2.4 Desktop, web, or both?

**Desktop only.** Cowork runs in the **Claude Desktop app for macOS and Windows** (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]**). Anthropic's reasoning: *"Claude Cowork runs on desktop, where most knowledge work is done"* (source: https://www.anthropic.com/product/claude-cowork — **[OFFICIAL]**). There is **no browser/web version of Cowork itself**; this is a hard contrast with Claude Code, which additionally has Claude Code on the web and IDE extensions. (Mobile messaging notifications are available to Pro/Max but the agent runs on the desktop.)

### 2.5 Plans/tiers and rollout status (as of mid-2026)

| Milestone | Date | Status | Source |
|---|---|---|---|
| Research preview launch | 2026-01-12 | macOS only, **Max subscribers only**, waitlist for other tiers | https://techcrunch.com/2026/01/12/anthropics-new-cowork-tool-offers-claude-code-without-the-code/ (community/press) |
| Enterprise/connectors expansion | 2026-02-24 | Wider enterprise release; connectors (Google Drive, Gmail, DocuSign, FactSet) + customizable plugins (financial analysis, engineering, HR) | CNBC 2026-02-24 (community/press; 403 to automated fetch, **full article verified via the Claude Chrome extension 2026-05-18** — see source #20) |
| Projects feature | ~2026-03-20 | Durable project workspaces added | https://cybersecuritynews.com/projects-feature-claude-cowork-desktop/ (community/press) |
| **General Availability** | **2026-04-09** | **GA for all paid plans, macOS + Windows**, plus 6 enterprise features | https://www.testingcatalog.com/anthropic-launches-claude-cowork-in-general-availability/ (community/press) |
| Claude for Small Business on Cowork | 2026-05-13 | 15 agentic workflows + 15 skills, SMB connectors | https://www.anthropic.com/news/claude-for-small-business (**[OFFICIAL]**) |

**Current availability (mid-May 2026):** **All paid plans — Pro, Max, Team, Enterprise — via the Claude Desktop app on macOS and Windows** (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork and https://www.anthropic.com/product/claude-cowork — **[OFFICIAL]**). Note: one official help article still describes Cowork as "available as a research preview for Pro, Max, Team, and Enterprise" while the product page and GA press say GA — see Confidence & gaps. **Advanced governance (RBAC, per-tool connector controls, group spend limits) is Team/Enterprise-only** (source: https://www.testingcatalog.com/anthropic-launches-claude-cowork-in-general-availability/ — community/press, 2026-04-09).

The six GA enterprise features (2026-04-09), per multiple community/press writeups: **(1) Role-Based Access Controls (RBAC)** — Enterprise, via SCIM/groups; **(2) Group Spend Limits**; **(3) Expanded Usage Analytics** (admin dashboard + Analytics API); **(4) Expanded OpenTelemetry support**; **(5) Zoom MCP connector**; **(6) Per-Tool Connector Controls** (sources: https://www.testingcatalog.com/anthropic-launches-claude-cowork-in-general-availability/ and https://9to5mac.com/2026/04/09/anthropic-scales-up-with-enterprise-features-for-claude-cowork-and-managed-agents/ — community/press; the same announcement also shipped a Claude Code policy/Bedrock-wizard update and Managed Agents public beta).

---

## 3. Cowork's execution model

### 3.1 Sandbox / VM

Cowork **runs agent-written code and shell commands inside an isolated virtual machine**, not on your host OS. Official help center: **"Shell commands and code Claude writes run inside an isolated virtual machine (VM), separate from your main operating system"** (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]**). This is the single most important architectural difference from Claude Code (which runs directly in your terminal with the launching user's full privileges).

### 3.2 File system access

- **Direct local file read/write, but folder-scoped.** *"Claude can read from and write to your local files without manual uploads or downloads"* but *"Claude can only read and write files in folders you've connected"* (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]**).
- Folders are granted via a GUI permission dialog with one-time or persistent ("Always Allow") grants (source: https://www.datacamp.com/tutorial/claude-cowork-tutorial — community).
- **Explicit permission required before permanently deleting files** (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]**).

### 3.3 Working directory / project notion

Cowork has a first-class **Projects** concept — the rough analogue of a Claude Code working directory + `CLAUDE.md` + memory: *"Group related tasks into separate workspaces with their own files, context, instructions, and memory"* (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]**). A project ties "one area of work tied to a folder, its instructions, and its evolving task history"; you can attach an existing folder or create one (source: https://cybersecuritynews.com/projects-feature-claude-cowork-desktop/ — community/press). Standalone (non-project) tasks also exist.

### 3.4 Long-running / async tasks and scheduling

- Cowork breaks complex work into subtasks and **coordinates parallel workstreams** (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]**) — the conceptual analogue of Claude Code subagents/`/batch`.
- **Scheduled/recurring tasks exist**: *"Create and save tasks that you can have Claude run on-demand, or automatically on a cadence of your choosing"* (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]**). This is the GUI analogue of Claude Code's `/schedule`/routines and `/loop`.
- **Constraint:** the desktop app must stay open while a task runs — *"If you close the app, your session will end"* (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]**). Unlike Claude Code's `/background` or web/cloud sessions, Cowork is **not a detached cloud agent**; it is a local desktop agent.

### 3.5 Code / terminal execution

Yes — Cowork can write and run code and shell commands, discover installed tools (e.g., LibreOffice, Ghostscript), and request permission before installing dependencies — **but all inside the VM sandbox** (sources: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]**; https://www.datacamp.com/tutorial/claude-cowork-tutorial — community). There is **no user-facing terminal**: the user does not type shell commands; they describe outcomes in natural language and approve plans.

### 3.6 Permission modes

Two modes, set in the GUI (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]**):
- **Ask before acting** — Claude pauses for approval (recommended for sensitive work).
- **Act without asking** — faster, riskier.

This is the GUI analogue of Claude Code's permission system (`/permissions`, allow/ask/deny rules, plan mode, auto-accept). Connector/MCP actions add a third granularity: **Always allow / Needs approval / Blocked** per permission category (source: https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities — **[OFFICIAL]**).

### 3.7 Memory

**Memory is project-scoped only**: *"Memory is supported within projects but is not retained across standalone Cowork sessions"* (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]**). Global standing instructions can be set in **Settings > Cowork** (tone/format/background guidance) — the loose analogue of personal `CLAUDE.md` / `~/.claude/CLAUDE.md` (source: same).

### 3.8 Usage cost note

*"Cowork consumes more of your usage allocation than chatting with Claude"* (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]**) — expected, since it is an agentic loop, same as Claude Code burning more than chat.

---

## 4. Connectors, MCP, and file I/O in Cowork

- **Connectors are MCP under the hood.** Anthropic: connectors *"work across Claude, Claude Desktop, Claude Code, and the API (via the MCP Connector)"* and are *"built on a technology called MCP (Model Context Protocol)"* (source: https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities — **[OFFICIAL]**).
- **Available connectors** include Google Drive, Gmail, Google Calendar (Google Workspace), Slack, DocuSign, FactSet, Zoom (MCP connector added at GA), plus SMB-oriented ones (QuickBooks, PayPal, HubSpot, Canva) shipped with Claude for Small Business (sources: https://support.claude.com/en/articles/10166901-use-google-workspace-connectors, https://www.anthropic.com/news/claude-for-small-business — **[OFFICIAL]**; CNBC 2026-02-24 — community/press). *Note: one community tutorial from earlier in the rollout said Gmail/Calendar/Drive connectors were "still in development" — by mid-2026 the official Google Workspace connector article lists them as available; treat the official article as current.*
- **Adding connectors (GUI):** in Cowork, click **Customize → Connectors → + → Browse connectors → Connect** (source: search-surfaced from https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork and connectors article — **[OFFICIAL]**).
- **Custom / remote MCP servers are supported in Cowork**: *"Custom connectors using remote MCP are available on Claude, Cowork, and Claude Desktop for users on free, Pro, Max, Team, and Enterprise plans"* — you supply a name + URL (+ optional OAuth). **Important architectural caveat:** *"Custom connectors connect to your MCP server from Anthropic's cloud, not from your local device. Your server must be reachable over the public internet"* (source: https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities — **[OFFICIAL]**). This differs from Claude Code, where local/stdio MCP servers run on the engineer's own machine.
- **File upload/download:** Cowork largely *doesn't need* upload/download — it reads/writes connected local folders directly (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]**).
- **Per-tool connector controls** (which MCPs are allowed, how often they prompt) are configurable; per-tool governance is a Team/Enterprise GA feature (sources: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]**; community GA writeups).

---

## 5. Claude Code vs. Claude Cowork — feature matrix

Legend: ✅ first-class · ⚠️ available but different/limited · 🔁 same capability, different surface · ❌ not available

| Dimension | Claude Code | Claude Cowork | Notes |
|---|---|---|---|
| **Primary surface** | Terminal CLI + VS Code/JetBrains extensions + Claude Code on web/desktop | Claude Desktop app, "Cowork" tab (macOS/Windows) | Cowork has **no web/CLI**; Code has no GUI-folder-picker model |
| **Target user** | Developers/engineers | Non-developer knowledge workers | Same engine, different audience (§2.2) |
| **Execution location** | Your real OS/shell, launching user's full permissions | **Isolated VM** for code/commands; folder-scoped FS | Biggest architectural delta (§3.1) |
| **File access** | Full filesystem (+ `--add-dir`, working dir) | Only **explicitly connected folders** | 🔁 Both can read/write local files |
| **Terminal / arbitrary shell** | ✅ Native, user-visible | ⚠️ Runs in sandbox VM, **no user terminal** | User describes outcomes, not commands |
| **Working dir / project** | CWD + `CLAUDE.md` + `.claude/` | **Projects** (folder + instructions + history + memory) | 🔁 Conceptually parallel |
| **Long-running / async** | `/background`, web/cloud sessions, routines | Scheduled/recurring tasks, but **app must stay open** | Cowork is **not** a detached cloud agent (§3.4) |
| **Built-in slash commands** | ~70 (`/init`, `/review`, `/mcp`, `/hooks`, `/plan`, etc.) | ❌ No operator command palette; `/` lists **Skills** | See §6 — the central conceptual point |
| **Custom commands** | `.claude/commands/*.md` / merged into Skills | 🔁 Via **Skills** packaged in plugins | Authoring model differs (file vs. uploaded ZIP) |
| **Skills** | ✅ `SKILL.md` + frontmatter, subagent/`context: fork`, dynamic `` !`cmd` `` injection, `allowed-tools`, etc. | ✅ Skills supported; built-in capability skills (xlsx/docx/pptx/pdf) + custom skills uploaded as ZIP | 🔁 Same open *Agent Skills* standard; Code adds CLI-only power features |
| **Memory** | `CLAUDE.md` (enterprise/personal/project/local), auto-memory, `/memory` | Project-scoped memory + global instructions in Settings | 🔁 Narrower, GUI-managed |
| **MCP servers** | Local (stdio) + remote, `/mcp`, OAuth | Remote/custom MCP via connectors UI; **runs from Anthropic cloud**, must be public | ⚠️ No local stdio MCP in Cowork |
| **Hooks / settings.json** | ✅ `.claude/settings.json`, hooks on tool events, managed settings | ❌ No user-facing hooks/settings.json; behavior via GUI + plugins/skills | Governance instead via org admin controls |
| **Subagents** | ✅ `/agents`, `.claude/agents/`, parallel/background | 🔁 Parallel workstreams internally; no user-authored subagent files exposed | Behavior present, authoring not user-exposed |
| **Plan mode** | ✅ `/plan`, plan-mode permission tier, `/ultraplan` | 🔁 "Review Claude's approach / approve the plan first" in GUI | Same concept, GUI gating |
| **Permissions / sandbox** | allow/ask/deny rules, `/permissions`, `/sandbox`, auto mode | "Ask before acting" / "Act without asking" + connector Always/Approval/Blocked | 🔁 GUI-expressed |
| **Git / GitHub** | ✅ Deep: commits, branches, `/review`, `/install-github-app`, `gh`, GitHub Actions, autofix PRs | ⚠️ Can operate on files in connected folders; **not** a Git-centric workflow | Cowork is document/data-centric, not VCS-centric |
| **Plugins / marketplace** | ✅ `/plugin`, marketplaces (skills+commands+agents+hooks+MCP) | ✅ Plugins bundle **skills + connectors + sub-agents**; Customize → Browse plugins; org marketplaces | 🔁 Same idea; Cowork plugin = no hooks, command→Skill |
| **Team/Enterprise governance** | Managed settings, policy controls, OpenTelemetry, Bedrock/Vertex | RBAC (Ent.), group spend limits, usage analytics, OpenTelemetry, per-tool connector controls, org plugin marketplaces, org-wide on/off toggle | Both have strong enterprise stories; mechanisms differ |
| **Output style** | Source code, commits, CI/CD | Spreadsheets, decks, PDFs, reports, organized files (community framing) | Reflects audience, not engine limits |
| **Pricing/availability** | Pro/Max/Team/Enterprise; CLI free to install | **All paid plans** (Pro/Max/Team/Enterprise), desktop only; advanced gov = Team/Ent | §2.5 |

---

## 6. Why a Claude Code user sees slash commands a Cowork user doesn't

This is the question most likely to confuse a Claude Code power user, so it deserves its own section. (A dedicated companion doc, `slash-commands-and-customization.md`, goes deeper.)

### 6.1 What Claude Code's slash commands actually are

Claude Code's `~70` slash commands are **CLI operator controls** — they "control Claude Code from inside a session… switch models, manage permissions, clear context, run a workflow" (source: https://code.claude.com/docs/en/commands — **[OFFICIAL]**). They fall into two categories (per that same official reference):

1. **Built-in commands** — *behavior coded into the CLI itself*: `/init`, `/mcp`, `/hooks`, `/agents`, `/permissions`, `/plan`, `/model`, `/compact`, `/clear`, `/resume`, `/config`, `/security-review`, `/review`, `/schedule`, `/background`, `/doctor`, etc.
2. **Bundled skills** — *prompt-based, Claude-orchestrated*, marked **[Skill]** in the reference: `/simplify`, `/batch`, `/debug`, `/loop`, `/claude-api`, `/fewer-permission-prompts`. These "use the same mechanism as skills you write yourself" (source: https://code.claude.com/docs/en/commands — **[OFFICIAL]**).

Crucially, even in Claude Code these are different things under the hood. The official Skills doc notes only *"a few built-in commands are also available through the Skill tool, including `/init`, `/review`, and `/security-review`. Other built-in commands such as `/compact` are not"* (source: https://code.claude.com/docs/en/skills — **[OFFICIAL]**).

### 6.2 Why most of them don't appear in Cowork

Three reasons, in order of importance:

1. **They are CLI plumbing with no GUI meaning.** Commands like `/compact`, `/context`, `/resume`, `/vim`, `/statusline`, `/tui`, `/teleport`, `/terminal-setup`, `/doctor`, `/release-notes` exist to manage a *terminal session*. Cowork has no terminal session to manage — context, sessions, and UI are handled by the desktop app's GUI. There is nothing to surface.
2. **The capability moved into the GUI.** The *function* of many commands still exists in Cowork, just not as a typed command:
   - `/init` + `/memory` (project setup + CLAUDE.md) → **Projects** with per-project instructions/memory + global instructions in **Settings > Cowork**.
   - `/mcp` (manage MCP/connectors) → **Customize → Connectors** UI.
   - `/permissions`, `/sandbox` → **"Ask before acting" / "Act without asking"** toggle + per-connector Always/Approval/Blocked.
   - `/plan` → **"review Claude's approach / approve the plan first"** GUI step.
   - `/agents` (subagents) → internal **parallel workstreams** (not user-authored).
   - `/schedule`, `/loop` → **scheduled/recurring tasks** in the Cowork UI.
   - `/plugin` → **Customize → Browse plugins** + org plugin marketplaces.
   - `/model`, `/config` → app **Settings**.
   - The official help center confirms this GUI mapping (project context, scheduled tasks, global instructions, permission modes, connector control) (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]**).
3. **The "command" abstraction Cowork *does* keep is the Skill.** This is the precise conceptual mapping: **Cowork surfaces invocable capability as Skills, not operator slash commands.** In Cowork you *"Type `/` or click the `+` button to see available Skills from your installed plugins"* (source: https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork — **[OFFICIAL]**). So the `/` key still does something — but it lists **Skills** (e.g., the built-in xlsx/docx/pptx/pdf capability skills, or skills from installed plugins), **not** `/init`-style CLI commands. This is consistent with the broader Skills model: Skills follow the open *Agent Skills* standard and are designed so *"Claude adapts skills to the surface it's in"* — chat, Cowork, or the Excel/PowerPoint add-ins (source: https://support.claude.com/en/articles/12512180-use-skills-in-claude — **[OFFICIAL]**).

### 6.3 The clean way to explain it

> *In Claude Code, `/` is the operator's control panel for a terminal session. In Cowork there is no terminal session, so `/` instead lists **Skills** (packaged capabilities/workflows). Everything a Claude Code user does with `/init`, `/mcp`, `/permissions`, `/plan`, `/schedule`, `/plugin` still exists in Cowork — it just moved into the desktop app's Projects, Connectors, permission toggles, scheduled tasks, and plugin browser instead of being typed. The agent's brain is the same; the cockpit is different.*

### 6.4 One sharp asymmetry to flag

The slash-command **authoring model differs**. In Claude Code a custom command is just a markdown file (`.claude/commands/x.md` or `.claude/skills/x/SKILL.md`) with frontmatter, picked up live, with CLI-only power features (`context: fork`, dynamic `` !`cmd` `` injection, `allowed-tools`, hooks-in-skills) (source: https://code.claude.com/docs/en/skills — **[OFFICIAL]**). In Cowork, custom Skills are **packaged as a ZIP and uploaded via Customize → Skills**, and *"Custom skills you upload are private to your individual account"* (source: https://support.claude.com/en/articles/12512180-use-skills-in-claude — **[OFFICIAL]**). Same standard, different distribution ergonomics — and Cowork users do not get `settings.json`/hooks-level determinism that a Claude Code power user leans on.

---

## 7. Practical guidance

- **Translate "command" → "Skill or GUI control."** When someone asks "how do I run a review/security scan/plan in Cowork like in Claude Code?", the answer is usually a Skill, a plugin, or a built-in GUI step — not a typed slash command.
- **Lead with the sandbox.** The single most reassuring (and accurate) enterprise message: agent-written code runs in an **isolated VM**, file access is **opt-in per folder**, and deletes require explicit approval (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]**). This is a deliberate contrast to Claude Code's full-host model.
- **Set expectations on "always-on."** Cowork is **not** a cloud/background agent — the desktop app must stay open. Anyone expecting fire-and-forget cloud automation should be pointed to Managed Agents / Agent SDK territory instead.
- **Governance pitch (Team/Enterprise):** org-wide on/off toggle, RBAC (Enterprise), group spend limits, usage analytics + Analytics API, OpenTelemetry to SIEM, per-tool connector controls, org plugin marketplaces with required/default/available/hidden tiers (sources: https://support.claude.com/en/articles/13455879-use-claude-cowork-on-team-and-enterprise-plans and https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork — **[OFFICIAL]**; GA feature list — community/press).
- **Two governance caveats to disclose proactively:** Cowork stores conversation history **locally** on users' machines and *"is not subject to Anthropic's standard data retention policies,"* and *"Cowork activity is not captured in the Compliance API at this time"* (source: https://support.claude.com/en/articles/13455879-use-claude-cowork-on-team-and-enterprise-plans — **[OFFICIAL]**). These are likely security-review questions; better to surface them first.

---

## Sources

1. https://www.anthropic.com/product/claude-cowork — **[OFFICIAL]** Anthropic Cowork product page. Backs positioning ("outcome over prompts"), audience, desktop-only, "all paid plans." No visible date; current as fetched 2026-05-18.
2. https://claude.com/blog/cowork-research-preview — **[OFFICIAL]** Anthropic blog "Cowork: Claude Code power for knowledge work." Backs "Claude Code power for knowledge work" framing, delegate/deliverable positioning, folder/connector control, plan-approval. References enterprise features dated 2026-04-08 and a March 2026 capability.
3. https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — **[OFFICIAL]** Claude Help Center "Get started with Claude Cowork." Backs nearly all execution-model facts: isolated VM, folder-scoped FS, Projects, project-scoped memory, scheduled tasks, app-must-stay-open, permission modes, usage cost, global instructions.
4. https://support.claude.com/en/articles/13455879-use-claude-cowork-on-team-and-enterprise-plans — **[OFFICIAL]** Help Center, Team/Enterprise Cowork governance: org-wide toggle, plugin marketplace tiers, OpenTelemetry to SIEM, local history + retention/Compliance-API caveats.
5. https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork — **[OFFICIAL]** Help Center, Cowork plugins: plugins bundle skills+connectors+sub-agents; `/` or `+` lists Skills; org marketplaces; all paid plans.
6. https://support.claude.com/en/articles/12512180-use-skills-in-claude — **[OFFICIAL]** Help Center, Skills in Claude/Cowork: built-in xlsx/docx/pptx/pdf capability skills, custom skills uploaded as ZIP, "adapts to the surface," plan availability.
7. https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities — **[OFFICIAL]** Help Center, connectors: MCP under the hood; cross-surface; custom remote MCP from Anthropic cloud (must be public); Always/Approval/Blocked.
8. https://support.claude.com/en/articles/10166901-use-google-workspace-connectors — **[OFFICIAL]** Help Center, Google Workspace connectors (Gmail/Calendar/Drive) availability (referenced via search summary).
9. https://www.anthropic.com/news/claude-for-small-business — **[OFFICIAL]** Anthropic news, 2026-05-13: Cowork as SMB platform, 15 workflows + 15 skills, QuickBooks/PayPal/HubSpot/Canva/Docusign connectors, plan-approve-or-run.
10. https://claude.com/blog/building-agents-with-the-claude-agent-sdk — **[OFFICIAL]** Anthropic engineering, orig. 2025-09-29: "the agent harness that powers Claude Code (the Claude Code SDK)… renamed the Claude Agent SDK." Backs shared-engine claim.
11. https://code.claude.com/docs/en/commands — **[OFFICIAL]** Claude Code commands reference: full ~70-command list, built-in vs. bundled-skill distinction, availability-varies note. (Redirected from docs.claude.com.)
12. https://code.claude.com/docs/en/skills — **[OFFICIAL]** Claude Code Skills doc: SKILL.md/frontmatter, Agent Skills open standard, commands-merged-into-skills, "few built-in commands available via Skill tool (`/init`,`/review`,`/security-review`); `/compact` not."
13. https://techcrunch.com/2026/01/12/anthropics-new-cowork-tool-offers-claude-code-without-the-code/ — community/press (TechCrunch, 2026-01-12). Backs Jan 12 launch, Max-only + waitlist, "built on the Claude Agent SDK," "sandboxed instance of Claude Code," folder-partition access.
14. https://www.testingcatalog.com/anthropic-launches-claude-cowork-in-general-availability/ — community/press (2026-04-09). Backs GA date, all-paid-plans, six enterprise features, advanced controls Team/Enterprise.
15. https://9to5mac.com/2026/04/09/anthropic-scales-up-with-enterprise-features-for-claude-cowork-and-managed-agents/ — community/press (2026-04-09). Corroborates GA enterprise features alongside Managed Agents beta.
16. https://cybersecuritynews.com/projects-feature-claude-cowork-desktop/ — community/press (~2026-03-20). Backs Projects = folder+instructions+task-history; MCP-compatible connectors.
17. https://www.datacamp.com/tutorial/claude-cowork-tutorial — community tutorial. Backs GUI folder permission dialog ("Always Allow"), sandboxed terminal/tool discovery, connectors browse path, native xlsx/docx/pptx/pdf skills. (Some connector-availability claims were rollout-stage; superseded by official articles.)
18. https://mattgeer.com/difference-between-claude-ai-cowork-and-code/ — community. Backs the "think / delegate / build" framing for Claude.ai vs. Cowork vs. Code.
19. https://help.apiyi.com/en/anthropic-claude-subscription-agent-sdk-billing-split-june-2026-en.html — community (treat as indicative). Backs the subscription-vs-Agent-SDK billing-pool split.
20. https://www.cnbc.com/2026/02/24/anthropic-claude-cowork-office-worker.html — community/press, CNBC "Anthropic updates Claude Cowork tool for the average office worker," 2026-02-24. Backs Feb 2026 enterprise/connectors expansion (Google Drive, Gmail, DocuSign, FactSet; financial-analysis/engineering/HR plugins), Kate Jensen / Peter McCrory quotes. *Originally HTTP 403 to automated fetch; **full article recovered via the Claude Chrome extension on 2026-05-18** — the connector/plugin enumeration is now verified verbatim, not search-summary.*

---

## Confidence & gaps

- **High confidence (official docs):** VM sandbox, folder-scoped file access, Projects + project-scoped memory, scheduled tasks, app-must-stay-open, permission modes, connectors = MCP, custom remote MCP runs from Anthropic cloud, Skills model and the "`/` lists Skills in Cowork" behavior, Cowork plugin composition, Team/Enterprise governance + the local-history/retention/Compliance-API caveats, the full Claude Code command list and built-in-vs-skill distinction. These are directly from `support.claude.com`, `anthropic.com`, and `code.claude.com`.
- **Plan/availability has a documented inconsistency.** The Cowork product page and GA press say **GA, all paid plans, macOS+Windows** (2026-04-09), but one official help article still phrases Cowork as a "research preview for Pro, Max, Team, and Enterprise." I treated the product page + GA coverage as current; help-center text may simply be lagging. Flag this if anyone cites the research-preview wording.
- **The "six GA enterprise features" list — resolved 2026-05-18.** The supposed `anthropic.com/news/claude-cowork` 404 was a non-issue: the canonical official GA post is **`claude.com/blog/cowork-for-enterprise`**, verified live via the Claude Chrome extension — *"Making Claude Cowork ready for enterprise,"* *"Cowork is now generally available on all paid plans,"* macOS + Windows, role-based access controls and group spend limits stated explicitly. RBAC/spend-limits/analytics/OpenTelemetry/Zoom-MCP/per-tool-controls are also corroborated by official Team/Enterprise help text. The full six-item *enumeration as a single list* still comes from community/press synthesis, but each item is now individually official-sourced. (The New Stack re-fetch was denied at the browser permission prompt; not needed — it was community-tier and its claims are now independently confirmed.)
- **Connector availability was rolling.** Earlier-rollout community material said Google Workspace connectors were "in development"; the official Google Workspace connectors help article lists Gmail/Calendar/Drive as available. I treated the official article as authoritative and current.
- **Billing-pool split** (subscription vs. Agent SDK credit) is community-sourced and may have changed; use only as background, not as a firm commitment.
- **Not independently verifiable here:** exact VM resource limits/timeouts, the precise internal subagent model in Cowork, and whether Cowork exposes any `settings.json`/hooks-equivalent to power users (official docs are silent; I inferred "no user-facing hooks/settings.json" from the absence of any such surface in the help center and the GUI-centric design — treat as a reasoned inference, not a documented absence).
- **CNBC Feb-2026 details — resolved 2026-05-18.** Originally a search summary (page blocked direct fetch with HTTP 403); the **full article was recovered via the Claude Chrome extension** and confirms the connector list (Google Drive, Gmail, DocuSign, FactSet) and plugin domains (financial analysis, engineering, HR) verbatim, alongside official sources 1 and 9. Now high confidence.

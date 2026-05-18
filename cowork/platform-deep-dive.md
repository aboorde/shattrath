# Claude Cowork — Platform Deep Dive (as of May 2026)

A platform/product-surface reference for a technically literate power user rolling Cowork out to a team. This focuses on the *product mechanics, surfaces, environment, connectors, and admin/governance reality*. Skills and Memory are covered only where they touch the platform surface (they have their own docs: `skills.md`, `memory.md`).

> Verification note: Researched live on 2026-05-18 against official Anthropic/Claude sources (help center `support.claude.com`, `claude.com/blog`, `anthropic.com/product/claude-cowork`, `privacy.claude.com`) plus reputable press, all dated/cited below. Cowork is evolving fast — treat anything tagged "beta" or "research preview" as a moving target.

---

## 1. What Cowork is

Claude Cowork brings Claude Code's agentic execution model to **non-coding knowledge work**, inside the **Claude Desktop app**. Instead of one-off chat responses, you hand Claude a goal and it plans and executes a multi-step task against your local files, applications, and connected services, returning a finished deliverable (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork ; https://www.anthropic.com/product/claude-cowork). Anthropic frames it as "built around the outcome" rather than the prompt, for "researchers, analysts, operations teams, legal professionals, and finance teams" (source: https://www.anthropic.com/product/claude-cowork).

The defining principle: "It completes tasks, but consequential decisions remain with the user." (source: https://www.anthropic.com/product/claude-cowork)

### Lifecycle / status timeline (official release notes)

| Date | Milestone |
|---|---|
| 2026-01-12 | Research preview launches — Claude Desktop, **macOS only**, **Max** plans (source: https://support.claude.com/en/articles/12138966-release-notes) |
| 2026-01-16 | Expanded to **Pro** plans (still macOS only) (source: https://support.claude.com/en/articles/12138966-release-notes) |
| 2026-02-24 | Plugin marketplace + admin controls for Team/Enterprise (source: https://support.claude.com/en/articles/12138966-release-notes) |
| 2026-02-25 | Scheduled / recurring tasks (source: https://support.claude.com/en/articles/12138966-release-notes) |
| 2026-03-17 | Mobile access — persistent agent thread via Claude for iOS/Android (Dispatch) (source: https://support.claude.com/en/articles/12138966-release-notes) |
| 2026-03-23 | Computer use ("Dispatch"/screen control) integration (source: https://support.claude.com/en/articles/12138966-release-notes) |
| **2026-04-09** | **General Availability** — macOS **and Windows**, all paid plans; six enterprise controls (source: https://claude.com/blog/cowork-for-enterprise) |

Cowork itself reached GA on 2026-04-09; **mobile Dispatch and computer use remain beta / research preview** for Pro and Max only as of May 2026 (source: https://support.claude.com/en/articles/13947068-assign-tasks-to-claude-from-anywhere-in-cowork ; https://support.claude.com/en/articles/14128542-let-claude-use-your-computer-in-cowork).

---

## 2. Interface & surfaces — how you start and structure work

### Entry point
Open the Claude Desktop app and switch from the **Chat** tab to the **Cowork** tab (Cowork sits alongside Chat and Code in the desktop app). Describe a task in natural language; Claude analyzes the request, creates a plan, and breaks complex work into subtasks (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).

### The unit of work: Tasks
Each piece of work is a **Task** (the Cowork analog of a chat conversation). Claude surfaces its reasoning and plan so you can follow along; you can delete any task from the desktop app (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).

### Projects (workspaces)
"Projects create persistent, self-contained workspaces with their own files, links, instructions, and memory." Each project gets its own instructions, files, and scheduled tasks; combined with Dispatch you can run multiple projects from a phone "without context bleeding between them" (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork ; https://claude.com/product/cowork). **Memory is supported within projects but NOT retained across standalone (non-project) Cowork sessions** (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).

### Scheduled / recurring tasks
Type `/schedule` in any task's input, or use the **Scheduled** sidebar → "+ New task". Cadence options: hourly, daily, weekly, weekdays, or manual/on-demand. The Scheduled sidebar lets you review runs, edit instructions/cadence, pause/resume, delete, or trigger on demand (source: https://support.claude.com/en/articles/13854387-schedule-recurring-tasks-in-claude-cowork).

> Gotcha: Scheduled tasks **only run while the desktop app is open and the computer is awake**. A missed run is *skipped* and auto-runs once the machine wakes / app reopens (with a notification + history entry). Anthropic explicitly says scheduled tasks are for "convenience automations," not mission-critical exact-time jobs (source: https://support.claude.com/en/articles/13854387-schedule-recurring-tasks-in-claude-cowork).

### Parallel / multi-task work
"For complex tasks, Claude may coordinate multiple sub-agents working simultaneously" and "coordinates multiple workstreams in parallel if appropriate." Long-running work is supported: "Work on complex tasks for extended periods without conversation timeouts or context limits interrupting your progress." (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork)

### Steering & stopping an in-progress agent
- **Permission modes:** "Ask before acting" (Claude pauses for approval — recommended for new/sensitive work) vs "Act without asking" (faster, riskier) (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).
- **Mid-task steering:** "You can jump in to course-correct or provide additional direction mid-task." Anthropic's framing: "Watch in real time or walk away." (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork ; https://claude.com/product/cowork)
- **Stopping:** delete a task at any time via the "Delete" option; closing the desktop app or sleeping the machine also halts active and scheduled tasks (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).

### Reviewing outputs / diffs / sharing
Claude surfaces its plan and reasoning for review before/while acting. Deliverables (spreadsheets, slides, docs) are written directly to your selected folders. **There is no diff-review UI like Claude Code's**; review is via the surfaced plan + the resulting files. **Sharing is a known gap: "No chat or artifact sharing: Sessions cannot be shared with others."** This applies to both tasks and projects (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).

---

## 3. Environment & capabilities

| Capability | Reality |
|---|---|
| **Code/shell execution** | Yes. "Shell commands and code Claude writes run inside an isolated virtual machine (VM), separate from your main operating system" — the VM runs locally on your computer (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork) |
| **Local filesystem** | Yes — direct read/write to folders **you explicitly grant**. "Claude can read from and write to your local files without manual uploads or downloads." Permanent deletion always requires explicit per-action approval (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork) |
| **Web browsing / fetch / search** | Yes. Caveat: "Network egress permissions don't apply to the web fetch or web search tools or MCPs." Web fetch "runs server-side and is limited to search results and URLs you've shared." Org owners can disable web search for Cowork+Chat (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork) |
| **Computer use (screen control)** | Beta, Pro/Max only. Claude takes screenshots and drives mouse/keyboard in your real apps. **"No sandbox between Claude and what's on your screen."** Per-app approval prompt; blocklist supported; trading/crypto/finance apps blocked by default (source: https://support.claude.com/en/articles/14128542-let-claude-use-your-computer-in-cowork) |
| **File/artifact creation** | Excel `.xlsx` (working formulas, VLOOKUP, conditional formatting, multi-tab), PowerPoint `.pptx`, Word `.docx`, PDF; further editable with Claude for Excel / PowerPoint. Standard artifacts (HTML/React, markdown, mermaid, SVG) also supported (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork ; https://claude.com/blog/create-files) |

**Tool priority order for computer use:** connectors first (Gmail/Slack), then browser navigation, then direct screen interaction as last resort for apps with no connector (source: https://support.claude.com/en/articles/14128542-let-claude-use-your-computer-in-cowork).

> The code/file sandbox protects your OS but **not your files** — "Code runs safely in an isolated space, but Claude can make real changes to your files." (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork)

---

## 4. Connectors & integrations

Connectors come in two forms; both work in Cowork unless noted:

- **Remote (web) connectors** — default; available on Claude, Cowork, Claude Desktop, Claude Mobile (source: https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities)
- **Desktop extensions** (local MCP, via MCPB) — Claude Desktop only; run "with the same permissions as any other program you run" (source: https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork)

### Verified named connectors

| Connector | Notes | Source |
|---|---|---|
| Google Workspace — **Gmail** | Search/read/draft emails; **cannot send on your behalf** | https://support.claude.com/en/articles/10166901-use-google-workspace-connectors |
| Google Workspace — **Drive** | Search/retrieve Docs, Sheets, Slides, PDFs, images, MS Office files | same |
| Google Workspace — **Calendar** | Search/retrieve events | same |
| **Slack** | Search channels/DMs/files; interactive connector for drafting/posting | https://support.claude.com/en/articles/13454812-use-interactive-connectors-in-claude |
| **GitHub** | Issues, PRs, read repo files, code search | https://support.claude.com/en/collections/15399129-connectors |
| **Microsoft 365** | Connector + dedicated security guide | https://support.claude.com/en/collections/15399129-connectors |
| **Zoom (MCP)** | Added at GA — AI Companion summaries, action items, transcripts, smart recordings | https://claude.com/blog/cowork-for-enterprise |
| Interactive connectors | Amplitude, Asana, Box, Canva, Clay, Figma, Hex, Slack — render live UIs in-conversation | https://support.claude.com/en/articles/13454812-use-interactive-connectors-in-claude |

There is also a **Connectors Directory** (claude.ai/connectors) with the full catalog; press references "200+ connectors" by late April 2026 (source: https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities ; releasebot — community aggregation, see Sources).

### Authorization & permission model
- Connectors mirror **your** existing access: "If someone can't access a specific file, channel, or record in the source system, the connector can't reach it from Claude either." (source: https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities)
- For Team/Enterprise, an **Owner/Primary Owner enables connectors org-wide**, but **each user still authenticates individually** (source: https://support.claude.com/en/articles/10166901-use-google-workspace-connectors).
- Per-action governance categories: **Always allow / Needs approval / Blocked**; admins can scope a connector read-only org-wide (e.g., read-only Gmail, block Drive edits) (source: https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities).

### MCP & custom connectors in Cowork
- Custom connectors via **remote MCP** are supported in Cowork for Free→Enterprise (Free limited to one) (source: https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities).
- "You control which MCPs you connect to Claude and how often they ask for permission." Anthropic explicitly warns to assess trust before extending access beyond defaults (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).

### Plugins (Cowork-specific bundling)
A **plugin** bundles Skills + connectors + sub-agents into one installable package. Library spans sales, finance, legal, marketing, HR, engineering, design, operations, data analysis (Anthropic shipped ~11 open-source plugins at launch). Install via Cowork → Customize → Browse plugins → Install (or upload custom files); plugins save locally (source: https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork ; https://www.anthropic.com/news/anthropic-raises-30-billion-series-g-funding-380-billion-post-money-valuation).

---

## 5. File handling

- **Access model:** no manual upload/download for local work — Claude reads/writes directly in folders you grant. Outputs land in the file system at locations Claude specifies on completion (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).
- **Created file types:** `.xlsx`, `.pptx`, `.docx`, `.pdf` + artifacts (source: https://claude.com/blog/create-files).
- **File size:** for Claude's create/upload feature generally, **30 MB per file** for both upload and download; outputs can be saved to Google Drive (source: https://claude.com/blog/create-files). Cowork's help article does not state a separate Cowork-specific cap (gap — see Confidence).
- **Persistence:** files persist on your machine (local). Conversation/agent **memory persists only within Projects**, not across standalone sessions (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork). On Team/Enterprise, conversation history is stored **locally on users' computers**, not centrally managed/exportable by admins (source: https://support.claude.com/en/articles/13455879-use-claude-cowork-on-team-and-enterprise-plans).

---

## 6. Collaboration & sharing

- **No native session/result sharing.** "Sessions cannot be shared with others." Sharing today is manual (share the produced file) (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).
- **Dispatch (mobile pairing):** pair phone ↔ desktop to send tasks from the Claude mobile app that execute on your computer. It is **one single continuous persistent thread** — "There's no way to start a new thread or manage multiple threads." Beta, **Pro/Max only**; desktop must be awake + app open (source: https://support.claude.com/en/articles/13947068-assign-tasks-to-claude-from-anywhere-in-cowork).
- **Team workspaces:** Projects are per-user workspaces, not shared multi-user team spaces; team-level consistency comes from **org-distributed plugins**, not shared sessions (source: https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork).

---

## 7. Security, privacy, governance & admin

### Layered safety model (official)
1. **Model training + classifiers:** RL to refuse malicious instructions; content classifiers "scan all untrusted content entering Claude's context" and flag prompt injection (source: https://support.claude.com/en/articles/13364135-use-claude-cowork-safely).
2. **Code execution isolation:** shell/code in an isolated local VM (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).
3. **Controlled file/network access:** only granted folders; permanent deletion needs explicit approval (source: https://support.claude.com/en/articles/13364135-use-claude-cowork-safely).
4. **Permission gating:** "Ask before acting" vs "Act without asking" — the latter "significantly increases the risk of prompt injection attacks" (source: https://support.claude.com/en/articles/13364135-use-claude-cowork-safely).

> Critical caveat: **computer use has NO sandbox** between Claude and the screen, and prompt-injection defenses are not absolute. Anthropic's general agent guidance recommends sandboxed/VM environments with no saved credentials and restricted network for computer-use-type agents (source: https://support.claude.com/en/articles/14128542-let-claude-use-your-computer-in-cowork ; https://www.anthropic.com/engineering/claude-code-sandboxing — Anthropic Claude Code sandboxing engineering post, reports ~1% prompt-injection success in internal testing).

### Admin / enterprise controls (Team & Enterprise)
Who can enable: **only org Owner / Primary Owner**, via Organization settings → Capabilities. The Cowork toggle is **org-wide all-or-nothing on Team**; **Enterprise can scope it by group/custom role (RBAC)** (source: https://support.claude.com/en/articles/13455879-use-claude-cowork-on-team-and-enterprise-plans).

Six GA enterprise controls (2026-04-09, source: https://claude.com/blog/cowork-for-enterprise):

| Control | Detail | Plan |
|---|---|---|
| **RBAC** | Group users (manual or SCIM from IdP); custom roles define accessible capabilities | Enterprise |
| **Group spend limits** | Per-team budgets from admin console | Team/Enterprise |
| **Usage analytics + Analytics API** | Cowork sessions, active users, per-user activity, skill/connector invocations, DAU/WAU/MAU | Team/Enterprise |
| **Expanded OpenTelemetry** | Emits events for tool/connector calls, files read/modified, skills used, manual-vs-auto approval; SIEM-compatible (Splunk, Cribl, Datadog, Elastic) | Team & Enterprise |
| **Zoom MCP connector** | Meeting intelligence into Cowork | All paid |
| **Per-tool connector controls** | Restrict actions within each MCP connector org-wide (e.g., read-only) | Team/Enterprise |

Plugin governance tiers (Team/Enterprise; Enterprise can apply per-group): **Installed by default / Available / Required (cannot uninstall) / Not available** (source: https://support.claude.com/en/articles/13455879-use-claude-cowork-on-team-and-enterprise-plans ; https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork).

### Audit / logging gap
**Cowork activity is NOT captured in the Compliance API** as of these docs. Observability for Cowork is via **OpenTelemetry + the analytics dashboard/Analytics API**, not the standard Compliance API audit log. Local conversation history on Team/Enterprise is not centrally exportable (source: https://support.claude.com/en/articles/13455879-use-claude-cowork-on-team-and-enterprise-plans). This is a meaningful governance limitation worth flagging to security/compliance stakeholders.

### Data retention & training policy
- **Consumer (Pro/Max):** delete a task → removed from history immediately, deleted from backend storage **within 30 days**. If model-improvement is opted in, chats may be retained de-identified up to **5 years** in training pipelines; opt-out keeps the 30-day default. Flagged (policy-violation) content retained up to 2 years (scores up to 7 years) (source: https://privacy.claude.com/en/articles/10023548-how-long-do-you-store-my-data ; https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).
- **Team/Enterprise (Commercial Terms):** "Customer prompts and responses aren't used to train our models by default. Retention is configurable." Custom retention requires Primary Owner/Owner (source: https://www.anthropic.com/product/enterprise ; https://privacy.claude.com/en/articles/10023548-how-long-do-you-store-my-data).
- Connector data (Gmail/Drive/Calendar) is stored on Anthropic servers and **not used to train models** (source: https://support.claude.com/en/articles/10166901-use-google-workspace-connectors).
- Claude Enterprise covers Claude Code **and** Cowork under one agreement (single procurement/security review); Enterprise includes SSO, SCIM, audit logs (chat), HIPAA-eligible (source: https://www.anthropic.com/product/enterprise ; https://www.anthropic.com/news/anthropic-raises-30-billion-series-g-funding-380-billion-post-money-valuation).

---

## 8. Plans, availability & limits

### Plan availability

| Plan | Cowork | Notes |
|---|---|---|
| Free | No | Paid only (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork) |
| Pro | Yes | + Dispatch mobile beta, computer use beta |
| Max (5x / 20x) | Yes | Highest usage allowances; Dispatch + computer use beta |
| Team | Yes | Org toggle is all-or-nothing; plugin governance; group spend/analytics |
| Enterprise | Yes | RBAC, per-group scoping, SCIM, OpenTelemetry, single agreement w/ Claude Code |

(source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork ; https://claude.com/blog/cowork-for-enterprise)

### Platform availability
**Claude Desktop only — macOS and Windows (arm64 + x64).** **Not on web.** Mobile (iOS/Android) can only *send* tasks via Dispatch beta; execution always happens on the paired desktop, which must be awake with the app open (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork ; https://support.claude.com/en/articles/13947068-assign-tasks-to-claude-from-anywhere-in-cowork).

### Usage limits
Cowork consumes quota **substantially faster than standard chat** (it reads files, plans, executes multi-step). Help-center guidance: batch related work into single sessions and use plain chat for simple tasks. Anthropic's official limits article does **not publish Cowork-specific numbers**; reputable community testing reports rough orders of magnitude (Pro ≈ 15–20 task batches/day, Max 5x ≈ ~30, Max 20x effectively unlimited; 5-hour rolling windows + weekly cap) — treat these as unofficial estimates, not Anthropic figures (source, official caveat: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork ; community estimates flagged in Sources).

---

## 9. Known limitations / gotchas

- **No session or artifact sharing** — collaboration is manual file handoff (official) (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).
- **Memory only within Projects**, not across standalone sessions — use project instructions/context files as workaround (official) (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).
- **Desktop must stay open + awake** — closing the app or sleep stops active *and* scheduled tasks; scheduled tasks aren't reliable for exact-time/mission-critical jobs (official) (source: https://support.claude.com/en/articles/13854387-schedule-recurring-tasks-in-claude-cowork).
- **Compliance API does not capture Cowork** — observability is OTel + analytics only; local history not centrally exportable (official) (source: https://support.claude.com/en/articles/13455879-use-claude-cowork-on-team-and-enterprise-plans).
- **Computer use has no sandbox** and is Pro/Max-only beta, *not* available on Team/Enterprise; prompt-injection guardrails "aren't absolute" (official) (source: https://support.claude.com/en/articles/14128542-let-claude-use-your-computer-in-cowork).
- **"Act without asking" materially raises prompt-injection risk** — recommend "Ask before acting" for rollout default (official) (source: https://support.claude.com/en/articles/13364135-use-claude-cowork-safely).
- **The local code VM protects the OS, not your files** — Claude makes real file changes; keep backups / dedicated working folders (official) (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).
- **Dispatch is a single non-branching thread**, Pro/Max beta only (official) (source: https://support.claude.com/en/articles/13947068-assign-tasks-to-claude-from-anywhere-in-cowork).
- **High quota burn** vs chat (official, qualitative) (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).

---

## Sources

1. https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — Primary official "Get started with Claude Cowork": surfaces, projects, VM/sandbox, permission modes, file access, sharing limitation, plans, platform, deletion/30-day retention. (Help center, current as of fetch 2026-05-18)
2. https://www.anthropic.com/product/claude-cowork — Official product page: positioning, safety principle ("consequential decisions remain with the user"), paid-plan availability.
3. https://claude.com/product/cowork — Official Cowork product page: scheduled tasks, projects, async model, Slack/Chrome integrations, plugin marketplace; lists feature dates (Enterprise Apr 8 2026, Dispatch Mar 23 2026, marketplace Feb 24 2026).
4. https://claude.com/blog/cowork-for-enterprise — Official GA announcement (2026-04-09): six enterprise features (RBAC, group spend limits, usage analytics/Analytics API, OpenTelemetry, Zoom MCP, per-tool connector controls), all-paid-plans + macOS/Windows.
5. https://claude.com/blog/cowork-research-preview — Official research-preview blog: positioning, "hand off a task / polished deliverable", Slack/Chrome/screen integration.
6. https://support.claude.com/en/articles/12138966-release-notes — Official release notes: dated lifecycle (preview 2026-01-12 macOS/Max; Pro 2026-01-16; scheduled 2026-02-25; mobile 2026-03-17; computer use 2026-03-23; marketplace 2026-02-24; GA 2026-04-09).
7. https://support.claude.com/en/articles/13854387-schedule-recurring-tasks-in-claude-cowork — Scheduled tasks: `/schedule`, cadences, skip/awake behavior, not for mission-critical.
8. https://support.claude.com/en/articles/13947068-assign-tasks-to-claude-from-anywhere-in-cowork — Dispatch/mobile: Pro/Max beta, single persistent thread, desktop-must-be-awake.
9. https://support.claude.com/en/articles/14128542-let-claude-use-your-computer-in-cowork — Computer use: no sandbox, per-app approval, blocklist, Pro/Max-only beta, not on Team/Enterprise.
10. https://support.claude.com/en/articles/13364135-use-claude-cowork-safely — Safety model: classifiers/prompt-injection scanning, permission modes, deletion protection, recommended practices.
11. https://support.claude.com/en/articles/13455879-use-claude-cowork-on-team-and-enterprise-plans — Admin/governance: Owner-only enablement, Team all-or-nothing vs Enterprise RBAC, plugin tiers, web-search toggle, Compliance API gap, local history.
12. https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork — Plugins: bundle skills+connectors+subagents, install flow, org marketplace governance, local MCP permission warning.
13. https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities — Connectors model: web vs desktop, permission mirroring, Always allow/Needs approval/Blocked, custom remote MCP.
14. https://support.claude.com/en/articles/10166901-use-google-workspace-connectors — Google Workspace specifics: Gmail/Drive/Calendar scope, "cannot send emails," per-user auth, not used for training.
15. https://support.claude.com/en/articles/13454812-use-interactive-connectors-in-claude — Interactive connectors list (Amplitude, Asana, Box, Canva, Clay, Figma, Hex, Slack), Cowork availability, permission reuse.
16. https://support.claude.com/en/collections/15399129-connectors — Connectors collection: confirms Slack, Google Workspace, GitHub, Microsoft 365 pre-built connector articles.
17. https://claude.com/blog/create-files — File creation: .xlsx/.pptx/.docx/.pdf, 30 MB up/down limit, Drive save, sandboxed compute.
18. https://privacy.claude.com/en/articles/10023548-how-long-do-you-store-my-data — Consumer retention: 30-day delete, 5-year training opt-in, 2yr/7yr policy-violation; commercial terms separate.
19. https://www.anthropic.com/product/enterprise — Enterprise: single Claude Code+Cowork agreement, no-training default, configurable retention, IdP/offboarding.
20. https://www.anthropic.com/news/anthropic-raises-30-billion-series-g-funding-380-billion-post-money-valuation — Official: ~11 open-source Cowork plugins, January 2026 launch context, HIPAA-eligible Enterprise.
21. https://www.anthropic.com/news/introducing-anthropic-labs (2026-01-13) — Confirms Cowork "launched as a research preview yesterday" (~2026-01-12).
22. https://www.anthropic.com/engineering/claude-code-sandboxing — Anthropic engineering: sandboxing rationale, ~1% internal prompt-injection success rate (Claude Code; informs Cowork agent-safety posture).
23. Community/press (clearly secondary, used only for cross-checks, not load-bearing claims): testingcatalog.com, thenewstack.io, pasqualepillitteri.it, releasebot.io aggregation (the "200+ connectors", per-plan task-batch estimates, 5-hour/weekly window figures originate here — flagged as unofficial).

## Confidence & gaps

- **High confidence (official, current):** surfaces (Tasks/Projects/Scheduled), local VM sandbox for code, file/permission model, "Ask before acting" vs "Act without asking", connector list & permission-mirroring, plugin governance tiers, the six GA enterprise controls, Compliance-API gap, sharing gap, memory-within-Projects-only, platform = desktop macOS/Windows, lifecycle dates from official release notes.
- **Canonical GA URL — resolved 2026-05-18.** Verified live via the Claude Chrome extension: the canonical official GA post is **`claude.com/blog/cowork-for-enterprise`** (source #4) — heading *"Making Claude Cowork ready for enterprise,"* *"Cowork is now generally available on all paid plans,"* macOS + Windows, role-based access controls + group spend limits. There is **no** separate `anthropic.com/news/claude-cowork` post; that 404 was a non-issue, not a missing source. Source counts of enterprise features ("four" vs "six") still differ in *secondary* coverage; the official blog (#4) frames the org-controls set and I used that.
- **Feb-2026 connectors/plugins (CNBC #23 cluster) — resolved 2026-05-18.** The CNBC article (HTTP 403 to automated fetch) was **fully recovered via the Claude Chrome extension**, confirming verbatim: 2026-02-24 connectors *Google Drive, Gmail, DocuSign, FactSet* and customizable plugins for *financial analysis, engineering, HR*. Those Feb facts are now high confidence, not press-summary.
- **Inferred / unofficial:** specific usage numbers (Pro ≈15–20 batches/day, Max tiers, 5-hour windows, weekly cap) and "200+ connectors" come from community/press, not an Anthropic numeric spec — Anthropic's official limits articles do **not** publish Cowork-specific figures. Treat as directional only.
- **Open gaps:** no official Cowork-specific file-size cap stated (the 30 MB figure is from the general file-creation feature, may or may not bind Cowork); no official published Cowork rate limits; computer-use prompt-injection rate (~1%) is Anthropic's Claude Code figure applied by analogy, not a Cowork-measured number.
- **Staleness risk:** Cowork is on a rapid release cadence (research preview Jan 2026 → GA Apr 2026, multiple monthly feature drops). Dispatch and computer use were still beta/preview and Pro/Max-only as of May 2026; re-check release notes (source #6) before any rollout decision, especially for Team/Enterprise computer-use availability and Compliance-API coverage, which are the most likely to change.

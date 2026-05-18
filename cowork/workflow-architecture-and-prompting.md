# Architecting Claude Cowork for a Team: Best-Practice Workflow Design and Prompt Engineering

A working guide for a technically literate power user who builds and owns the skills, prompts, and workflows that a team runs on Claude Cowork. It translates Anthropic's official prompt-engineering, agent, and context-engineering guidance into concrete Cowork architecture decisions, then applies them to real-world workflows (recurring status review, review-deck prep, risk scoring, request triage, notes → system of record, stakeholder comms), and ends with iteration, observability, and team governance.

Marking convention: **[ANTHROPIC-OFFICIAL]** = anthropic.com / claude.com / platform.claude.com / support.claude.com / Anthropic Academy/Skilljar. **[COMMUNITY]** = reputable practitioner content, clearly secondary.

---

## 1. What Cowork actually is, and the mental model that should drive your architecture

Cowork is Claude's agentic mode in the Claude desktop app: you describe an outcome, Claude plans and executes multi-step work against your local files, folders, and connected apps, and you steer along the way. It is positioned as "Claude Code power for knowledge work" — taking on a full task end-to-end rather than answering one question at a time `(source: https://claude.com/product/cowork)` `(source: https://www.anthropic.com/product/claude-cowork)`. It went from research preview (late January 2026) to general availability (April 9, 2026), with the GA specifically closing the enterprise governance gap CIOs needed `(source: https://thenewstack.io/anthropic-takes-claude-cowork-out-of-preview-and-straight-into-the-enterprise/)` `(source: https://pasqualepillitteri.it/en/news/755/anthropic-managed-agents-cowork-ga-april-9-2026)` [COMMUNITY, reporting on Anthropic announcements].

The single most important architectural idea, straight from Anthropic's agent guidance: **build the simplest system that works, and only add complexity when it earns its place.** "Success in the LLM space isn't about building the most sophisticated system. It's about building the *right* system for your needs" `(source: https://www.anthropic.com/research/building-effective-agents)` [ANTHROPIC-OFFICIAL]. The corollary for context engineering: **find the smallest set of high-signal tokens that maximize the likelihood of your desired outcome** `(source: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)` [ANTHROPIC-OFFICIAL].

Anthropic's official framing of how you personalize Cowork is a three-level ladder, and your architecture should follow it in order `(source: https://claude.com/resources/tutorials/customize-claude-cowork)` [ANTHROPIC-OFFICIAL]:

1. **Context and tools** — connectors + instructions (global and project-level). The background rules and live data Claude always has.
2. **Process capture** — Skills. Repeatable procedures, captured once, triggered automatically.
3. **Bundle and share** — Plugins. Skills + connectors + sub-agents packaged for a team, distributable by admins.

> "Most teams progress sequentially: connectors and instructions first → skills after repeating tasks → plugins for team standardization through admins" `(source: https://claude.com/resources/tutorials/customize-claude-cowork)` [ANTHROPIC-OFFICIAL].

Resist the urge to start by writing twelve skills. Start with a few connectors, tight instructions, and one well-scoped folder; promote to skills only after you've repeated a task enough to know its real shape.

---

## 2. The component map: what belongs where (skill vs memory vs connector vs instructions vs plugin vs ad-hoc prompt)

This is the decision you'll make most often. Get it wrong and you get context bloat, mis-triggered skills, and an agent that ignores your rules. Anthropic's own definitions:

| Component | What it is (official) | Lifetime / scope | Put this here |
|---|---|---|---|
| **Ad-hoc prompt** | A conversation-level instruction for a one-off task | Single task | "When you need Claude to do something once, just prompt well" `(source: https://claude.com/resources/tutorials/teach-claude-your-way-of-working-using-skills)` [ANTHROPIC-OFFICIAL] |
| **Instructions (global / project)** | Background rules Claude follows regardless of task; global = every session, project = that project | Persistent, always loaded | Short, universal rules: tone, do-not-touch folders, "never email a stakeholder without my approval." "Instructions apply to every task" `(source: https://claude.com/resources/tutorials/customize-claude-cowork)` `(source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork)` [ANTHROPIC-OFFICIAL] |
| **Skill** | A filesystem resource (`SKILL.md` + optional bundled files/scripts) giving Claude domain expertise; loads on-demand, triggers automatically | Persistent, loaded only when relevant | A repeatable *procedure* with steps, templates, reference data. "Skills can contain far more detail. They can include extensive instructions, complete reference libraries and detailed frameworks that would otherwise clutter custom instructions" `(source: https://claude.com/resources/tutorials/teach-claude-your-way-of-working-using-skills)` [ANTHROPIC-OFFICIAL] |
| **Memory (project)** | Per-project persistent summary Claude maintains across chats | Project-scoped; updated ~every 24h, capturing the useful bits | Evolving project/workstream facts that accumulate over time; not procedures `(source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork)` [ANTHROPIC-OFFICIAL]; `(source: https://karozieminski.substack.com/p/claude-cowork-guide-plugins-memory-sub-agents-tips)` [COMMUNITY] |
| **Connector (MCP)** | Live read/write integration to an external app (Salesforce, Slack, Gmail, Google Drive, Jira, Notion, GitHub, etc.) | Persistent auth across sessions | The live system of record. Derive from it; never transcribe a stale copy into a skill `(source: https://claude.com/resources/tutorials/customize-claude-cowork)` `(source: https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork)` [ANTHROPIC-OFFICIAL] |
| **Plugin** | A bundle of skills + connectors + sub-agents, installable/shareable, admin-distributable via marketplaces | Team-distributable unit | The packaged, governed delivery vehicle for a whole role's setup `(source: https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork)` [ANTHROPIC-OFFICIAL] |

### The decision rules

- **Once → prompt. Repeatedly → skill.** The trigger to make a skill is "you've figured out how something should be done, and find yourself explaining it to Claude multiple times" `(source: https://claude.com/resources/tutorials/teach-claude-your-way-of-working-using-skills)` [ANTHROPIC-OFFICIAL].
- **Procedure → skill. Fact that accumulates → memory. Universal rule → instructions.** Skills are *how to do a thing*; project memory is *what we've learned about this workstream over time*; instructions are *the rules that always apply*. A skill is task-specific and activates only when relevant; instructions apply universally `(source: https://claude.com/resources/tutorials/teach-claude-your-way-of-working-using-skills)` [ANTHROPIC-OFFICIAL].
- **Live data → connector, never a copied list.** The system of record owns the project list, key metrics, and milestone dates; the ticketing system owns request state. Reference them through connectors at runtime. This is just-in-time context retrieval: maintain lightweight identifiers and load data dynamically rather than pre-loading or duplicating it `(source: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)` [ANTHROPIC-OFFICIAL]. A copied project list rots; a connector query doesn't.
- **Skill works everywhere; project is for accumulated context.** "Skills work everywhere. Create a Skill once, and it's available in any conversation… Projects suit work with accumulated context over time" `(source: https://claude.com/resources/tutorials/teach-claude-your-way-of-working-using-skills)` [ANTHROPIC-OFFICIAL]. So: per-project context lives in a Project (memory + folder); the *review-deck-prep method* lives in a Skill used inside that Project.

---

## 3. How Skills work — the architecture you must design around

A Skill is a directory with a required `SKILL.md` (YAML frontmatter + body) plus optional bundled files and scripts. It loads in three progressive-disclosure levels `(source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)` [ANTHROPIC-OFFICIAL]:

| Level | When loaded | Token cost | Content |
|---|---|---|---|
| 1: Metadata (`name`, `description`) | Always, at startup, injected into system prompt | ~100 tokens/skill | Discovery info — Claude only knows the skill *exists* and *when to use it* |
| 2: `SKILL.md` body | When the skill triggers | Under ~5k tokens; keep body **under 500 lines** | The procedure/workflow |
| 3: Bundled files & scripts | As referenced | Effectively unlimited; scripts execute without loading their code into context | Reference docs, templates, deterministic scripts |

Design implications for a skill library:

- **The `description` is the single highest-leverage field you write.** Claude picks the right skill from potentially 100+ using only the descriptions. It must state *what it does* and *when to use it*, in **third person**, with concrete trigger terms `(source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)` [ANTHROPIC-OFFICIAL]. Claude has "a measured tendency to under-trigger skills," so write descriptions slightly "pushy" `(source: https://generativeprogrammer.com/p/skill-authoring-patterns-from-anthropics)` [COMMUNITY, summarizing Anthropic best-practices].
  - Good: `Prepares a recurring business review brief for a key project: pulls usage and status data, summarizes adoption trends, and drafts the review deck outline. Use when the user asks to prep a business review, build a review deck, or get ready for a stakeholder review meeting.`
  - Bad: `Helps with stakeholder meetings.`
- **Concise is key — the context window is a public good.** Default assumption: Claude is already very smart; only add context it doesn't have. Challenge every paragraph: "Does this justify its token cost?" `(source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)` [ANTHROPIC-OFFICIAL].
- **Progressive disclosure by domain.** Put the method in `SKILL.md`; push schemas, segment definitions, comms templates, and rubrics into separate one-level-deep reference files (`reference/status-scoring.md`, `templates/review-deck-outline.md`). Keep references **one level deep from SKILL.md** — Claude may only partially read deeply nested files (`head -100`), causing incomplete info `(source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)` [ANTHROPIC-OFFICIAL]. For reference files >100 lines, add a table of contents.
- **Set the right degrees of freedom.** High freedom (prose steps) when many approaches are valid (e.g., "how to summarize a project"); low freedom (exact script/command, "do not modify") when the operation is fragile or must be consistent (e.g., a system-of-record write, a redaction step) `(source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)` [ANTHROPIC-OFFICIAL].
- **Explain the *why*, not just the rule.** "State the rule, then explain why so Claude can generalise to cases the skill did not spell out." The reasoning becomes the rubric for situations you didn't anticipate `(source: https://generativeprogrammer.com/p/skill-authoring-patterns-from-anthropics)` [COMMUNITY, summarizing Anthropic]; this mirrors official prompt guidance that "Claude is smart enough to generalize from the explanation" `(source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Naming: gerund form, specific.** `preparing-review-decks`, `triaging-requests`, `assessing-risk` — not `helper`, `utils`, `stuff`. Consistent naming aids discovery and team discussion `(source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)` [ANTHROPIC-OFFICIAL]. (Note: `name` is lowercase/hyphens only, ≤64 chars, no "anthropic"/"claude"; `description` ≤1024 chars `(source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)` [ANTHROPIC-OFFICIAL].)
- **Avoid time-sensitive content and offering too many options.** Use an "Old patterns" `<details>` block for deprecated steps; give one default with an escape hatch, not five libraries to choose from `(source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)` [ANTHROPIC-OFFICIAL].
- **Sharing scope is not symmetric — know it before you design rollout.** In claude.ai each user uploads skills individually (no central admin/org distribution); the API is workspace-wide; Claude Code is filesystem-based. Cowork's team-distribution path is **Plugins via admin marketplaces** `(source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)` `(source: https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork)` [ANTHROPIC-OFFICIAL].

---

## 4. Prompt engineering for an agentic product — the rules that change Cowork behavior

Cowork runs on Claude's latest models (Opus/Sonnet 4.x family). These models are **more literal, more concise, more autonomous, and use tools less by default** than prior generations — every one of those shifts changes how you should write skills and task prompts `(source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].

### 4.1 Core principles (apply these in every skill and task prompt)

- **Be clear and direct; treat Claude as a brilliant new employee with no context on your norms.** Golden rule: "Show your prompt to a colleague with minimal context… If they'd be confused, Claude will be too" `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Add the *why* behind instructions** — Claude generalizes from it. ("This brief is read aloud to the exec team, so keep sentences short" beats "be concise.") `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Use 3–5 examples (multishot), wrapped in `<example>`/`<examples>` tags.** Examples are "the 'pictures' worth a thousand words" — the most reliable way to steer format/tone. Make them relevant, diverse, structured `(source: …claude-prompting-best-practices)` `(source: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)` [ANTHROPIC-OFFICIAL].
- **Structure with XML tags** (`<instructions>`, `<context>`, `<project_data>`, `<output_format>`) so Claude unambiguously separates rules from data `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Give Claude a role.** Even one sentence in instructions: "You are a senior analyst preparing executive-facing analyses" focuses tone and judgment `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Tell Claude what to do, not what not to do** (positive instructions and positive examples outperform negative ones, especially for verbosity control) `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Be explicit when you want action vs. analysis.** "Can you suggest changes" → Claude may only suggest. "Update the status tracker with these values" → Claude acts. Use a `<do_not_act_before_instructions>` block in a skill where you want research-then-confirm, or `<default_to_action>` where you want autonomy `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Long inputs go at the top.** When stuffing transcripts/reports into a prompt, put the long data *above* the instructions/query — improves quality up to ~30% on multi-doc inputs. Ask Claude to **quote relevant passages first** (in `<quotes>`) before analyzing, to cut through noise `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Define success criteria explicitly** for any research/synthesis task ("a successful answer names the top 3 risk drivers with the data point behind each") `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Ask Claude to self-check.** "Before you finish, verify every ARR figure against the connector data and flag any you couldn't confirm." This reliably catches errors `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].

### 4.2 Behaviors specific to the current models that you must tune

- **Literalism: state scope explicitly.** The model won't silently generalize one instruction to all items. If a formatting rule applies to every project section, say "Apply this to every section, not just the first" `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Don't over-prompt tool/skill triggering.** Newer models are *more* responsive to the system prompt and will over-trigger on aggressive language. Replace "CRITICAL: you MUST use this skill when…" with normal "Use this skill when…". If a skill or web search *under*-triggers, describe clearly *why and when* it should fire rather than shouting `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Tool use is lower by default; raise it with effort or explicit instruction.** If Cowork isn't pulling from a connector or searching when it should, explicitly tell it when/why to use that tool `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Curb overengineering.** These models tend to add unrequested files/abstractions. For deliverables, add a scope clamp: "Only produce the requested brief. Don't create extra helper files, don't refactor the template, don't add sections that weren't asked for" `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Verbosity is adaptive.** If your review brief needs a fixed shape, prescribe it (template + "concise, focused responses; skip non-essential context"); positive examples beat "don't ramble" `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Thinking is adaptive and steerable.** For multi-step reasoning (risk scoring) you can prompt "reason through the risk factors before scoring." If latency matters and you don't need deep reasoning, say so. Note the model is sensitive to the word "think" — "consider/evaluate/reason through" work as alternatives `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Subagents: fewer by default, steerable.** Tell Cowork when fan-out is wanted ("spawn subagents to summarize these 30 requests in parallel") and when not ("do single-project work directly, don't delegate") `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL]. A community test reported 10 files going from ~30 min sequential to ~4 min with explicit parallel sub-agents `(source: https://karozieminski.substack.com/p/claude-cowork-guide-plugins-memory-sub-agents-tips)` [COMMUNITY].
- **Balance autonomy and safety with an explicit reversibility clause.** Add to instructions/skill: take local reversible actions freely (drafting files), but **ask before** anything hard to reverse or visible to others — sending a stakeholder email, posting to Slack, writing to the system of record, deleting files `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL]. This pairs with Cowork's "Ask before acting" mode (see §6).

---

## 5. Agentic workflow design — picking the right pattern per workflow

Anthropic's distinction: **workflows** = predefined paths; **agents** = model dynamically directs itself. Use a workflow shape for well-defined, predictable tasks; use agentic latitude for open-ended ones — and remember agents trade latency/cost for capability `(source: https://www.anthropic.com/research/building-effective-agents)` [ANTHROPIC-OFFICIAL]. The patterns, translated to Cowork skills:

| Pattern (official) | When | Application |
|---|---|---|
| **Prompt chaining** (sequential steps, each on prior output) | Decomposable into fixed subtasks; trade latency for accuracy | Notes → structured fields → system-of-record write → confirmation summary. Self-correction chain (draft → review-against-rubric → revise) is the most common useful chain `(source: …building-effective-agents)` `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL] |
| **Routing** (classify, send to specialized path) | Distinct input types needing specialized handling | Request triage: classify (bug / how-to / billing / risk-signal) → route to the matching sub-procedure |
| **Parallelization** (sectioning / voting) | Independent subtasks, or multiple perspectives | Summarize 40 support requests or 25 call transcripts via sub-agents in parallel |
| **Orchestrator–workers** | Subtasks can't be predicted up front | "Prep review decks for my whole portfolio": orchestrator enumerates projects from the system of record, spawns a worker per project |
| **Evaluator–optimizer** | Clear eval criteria + iterative refinement adds value | Draft a high-stakes stakeholder email → evaluate against your comms rubric → refine until it passes |

Design principles for every workflow you build `(source: https://www.anthropic.com/research/building-effective-agents)` [ANTHROPIC-OFFICIAL]:

1. **Simplicity** — the smallest pipeline that works.
2. **Transparency** — make the plan visible (Cowork shows it; keep skill steps explicit and checklist-able).
3. **Well-documented, well-tested tools/connectors** — invest in the agent-computer interface as much as the prompt.

Use Cowork's planning step as a real control surface: a skill should produce a checklist Claude copies into the response and ticks off; "clear steps prevent Claude from skipping critical validation" `(source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)` [ANTHROPIC-OFFICIAL].

### Context engineering for long-running work

For portfolio-scale tasks, apply Anthropic's long-horizon techniques `(source: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)` [ANTHROPIC-OFFICIAL]:

- **Context rot is real:** more tokens → worse recall, before any hard limit. Keep each task's context tight; don't dump the whole system of record into one prompt.
- **Just-in-time retrieval:** hold identifiers (project IDs, file paths, saved queries) and pull data when needed, via connectors — not pre-loaded copies.
- **Structured note-taking / memory:** for multi-project runs, have the skill write a running progress/state file (e.g., `outputs/review-run-status.md`) so a fresh context window can resume. Use structured formats (JSON/Markdown table) for status, prose for notes.
- **Sub-agent isolation:** delegate detailed per-project digging to sub-agents that return a ~1–2k-token distilled summary, keeping the lead context clean.
- **System-prompt "right altitude":** instructions should be specific enough to guide but not brittle hardcoded logic — start minimal, add only what observed failures require.

### Connector design (when you or an admin build/configure MCP connectors)

From "Writing effective tools for AI agents" `(source: https://www.anthropic.com/engineering/writing-tools-for-agents)` [ANTHROPIC-OFFICIAL]:

- **Consolidate around workflows, not API endpoints.** Prefer a `get_project_status(project_id)` tool that returns the synthesized picture over forcing the agent to chain `list_projects` + `get_usage` + `get_requests`.
- **Namespace tools** (`sor_projects_search`, `sor_opportunities_search`) so the agent doesn't pick the wrong one.
- **Return meaningful context** — semantic names over raw UUIDs improves precision.
- **Token efficiency** — pagination/filtering/sensible truncation; offer concise vs. detailed response formats.
- **Bloated tool sets are the #1 failure mode:** "If a human engineer can't definitively say which tool should be used in a given situation, an AI agent can't be expected to do better" `(source: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)` [ANTHROPIC-OFFICIAL]. Enable only the connectors a workflow needs.

---

## 6. Cowork operational architecture — projects, folders, memory, permissions, scheduling

Official Cowork mechanics you must design around `(source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork)` `(source: https://claude.com/resources/tutorials/customize-claude-cowork)` [ANTHROPIC-OFFICIAL]:

- **Projects** group related tasks into a self-contained workspace with their **own files, links, instructions, context, and memory** — the right container for a project, a segment, or a recurring program.
- **Instructions are two-tier:** Global (Settings → Cowork → Global instructions, every session) and Project-level (supplements global). Keep global short and universal; push specifics to projects/skills.
- **Folder instructions** attach context to a selected local folder; Claude can update them itself during a session.
- **Controlled file & network access:** Claude can only read/write files in folders you connect. **Grant the minimum** — a dedicated project folder, not all of Documents `(source: https://karozieminski.substack.com/p/claude-cowork-guide-plugins-memory-sub-agents-tips)` [COMMUNITY], consistent with official least-access design `(source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork)` [ANTHROPIC-OFFICIAL].
- **Two approval modes:** "Ask before acting" (Claude pauses for approval on each action — use for anything stakeholder-facing or system-of-record-writing) and "Act without asking" (faster, riskier — reserve for sandboxed/internal-only runs). Claude always asks before permanently deleting files `(source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork)` [ANTHROPIC-OFFICIAL].
- **Memory is project-scoped, not global.** Within a Project you get persistent memory across chats; Claude maintains a separate memory summary per project plus one for non-project chats, updating ~every 24h with the useful bits. There is **no automatic cross-session memory outside projects** — by default standalone Cowork sessions start fresh `(source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork)` [ANTHROPIC-OFFICIAL]; `(source: https://karozieminski.substack.com/p/claude-cowork-guide-plugins-memory-sub-agents-tips)` [COMMUNITY]. **Architecture consequence:** anything that *must* persist precisely (a milestone date, a commitment made to a stakeholder) should be written to an explicit file in the project folder, not trusted to the 24h auto-summary.
- **Scheduled tasks:** `/schedule` in any task creates recurring work, but **only runs while the computer is awake and the desktop app is open** `(source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork)` [ANTHROPIC-OFFICIAL]. Don't architect a "Monday 6am status report" assuming server-side execution.

### Recommended project skeleton for a portfolio of work

```
Project: [Segment or Workstream]
├── Project instructions  → role, tone, approval rules, "never email without approval"
├── Connectors            → system of record, ticketing, calendar, email, Slack (only what's needed)
├── /references/          → status-scoring rubric, segment defs, comms style guide
├── /templates/           → review-deck outline, risk memo, exec-summary format
├── /workstreams/<ws>/    → per-workstream working folder + notes
├── /outputs/             → generated deliverables
└── /run-status/          → progress/state files for multi-workstream runs
```
This mirrors the community-tested folder discipline (`references/`, `templates/`, `outputs/`, status files) `(source: https://karozieminski.substack.com/p/claude-cowork-guide-plugins-memory-sub-agents-tips)` [COMMUNITY] and Anthropic's structured-note-taking guidance `(source: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)` [ANTHROPIC-OFFICIAL].

---

## 7. Concrete workflow blueprints

Each blueprint specifies the component split (skill / memory / connector / instructions) and the sourced principle behind each design choice.

### 7.1 Recurring status review summary

- **Connectors:** system of record (usage, key metrics, stage), ticketing (open/aged requests), product analytics if available — *live data, never copied* `(source: …effective-context-engineering…)` [ANTHROPIC-OFFICIAL].
- **Skill `summarizing-project-status`:** body = the method (pull metrics → compare to segment baseline → classify status → name top 3 risks/wins → recommend next action). Status-scoring thresholds live in `references/status-scoring.md` (one level deep) so they're maintained in one place and not duplicated per run `(source: …agent-skills/best-practices)` [ANTHROPIC-OFFICIAL].
- **Output template** in `templates/` with a strict structure (Exec summary → Status verdict → Evidence → Recommended action). Use the strict-template pattern; exec-facing output needs consistency `(source: …agent-skills/best-practices)` [ANTHROPIC-OFFICIAL].
- **QA step in the skill:** "Quote the exact data point behind every claim; flag any metric you could not retrieve." Grounding-in-quotes + self-check `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Memory:** the *narrative* ("this project has been amber two quarters running, exec sponsor left in Q1") belongs in project memory/notes; the *scoring method* belongs in the skill.

### 7.2 Review-deck prep

- **Pattern:** orchestrator–workers for a whole portfolio ("prep review decks for all my key projects") — enumerate projects from the system of record, sub-agent per project `(source: …building-effective-agents)` [ANTHROPIC-OFFICIAL].
- **Skill `preparing-review-decks`:** workflow with a copy-able checklist (pull data → adoption trend → outcomes vs. stated goals → risks → expansion opportunities → draft deck outline). Checklists stop step-skipping `(source: …agent-skills/best-practices)` [ANTHROPIC-OFFICIAL].
- **Reference files:** `templates/review-deck-outline.md`, `references/value-narrative-framework.md`. Domain-organized progressive disclosure so a single-project run doesn't load the whole library `(source: …agent-skills/best-practices)` [ANTHROPIC-OFFICIAL].
- **Sub-agent return contract:** each project sub-agent returns a ≤2k-token distilled brief to keep the lead context clean `(source: …effective-context-engineering…)` [ANTHROPIC-OFFICIAL].
- **Human-in-the-loop:** deck-outline stage pauses for user approval before any slide tool runs (Ask-before-acting) `(source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork)` [ANTHROPIC-OFFICIAL].

### 7.3 Risk scoring assessment

- **Pattern:** prompt chaining with explicit reasoning + evaluator step. Chain: gather signals (usage trend, support sentiment, exec engagement, contract value, competitive mentions) → reason through risk → score → recommend play.
- **Prompt design:** "Reason through each risk factor before assigning the score" (adaptive thinking, multi-step) and "develop competing hypotheses, track confidence" for ambiguous projects `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Skill `assessing-risk`:** scoring rubric in `references/risk-rubric.md`; the *why* of each factor stated so Claude generalizes to factors not enumerated `(source: https://generativeprogrammer.com/p/skill-authoring-patterns-from-anthropics)` [COMMUNITY, summarizing Anthropic].
- **Verifiable intermediate output:** produce a structured `risk-assessment.json` (factors, scores, confidence) before writing the narrative memo — plan-validate-execute catches bad inputs early `(source: …agent-skills/best-practices)` [ANTHROPIC-OFFICIAL].

### 7.4 Request triage

- **Pattern:** routing — classify each request, route to a specialized sub-procedure. Routing enables "separation of concerns and more specialized prompts" `(source: …building-effective-agents)` [ANTHROPIC-OFFICIAL].
- **Connector:** ticketing system (read), Slack (notify), system of record (link to recurring status review).
- **Skill `triaging-requests`:** classification taxonomy with 3–5 labeled few-shot examples per category (multishot is the most reliable steering for classification) `(source: …claude-prompting-best-practices)` `(source: https://www.anthropic.com/news/prompt-engineering-for-business-performance)` [ANTHROPIC-OFFICIAL].
- **Parallelize** across a backlog via sub-agents; ask explicitly for parallel fan-out `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Scope clamp:** "Classify and route only. Do not reply to stakeholders or change request status without approval" — autonomy/safety reversibility clause `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].

### 7.5 Meeting-notes → system of record

- **Pattern:** prompt chaining: raw notes → extract structured fields (attendees, decisions, action items, sentiment, next steps) → map to system-of-record schema → write → confirm.
- **Connector:** system of record (write) + calendar/notes source.
- **Skill `logging-meeting-notes-to-system-of-record`:** the field-mapping is **low freedom** (exact mapping, "do not invent fields"); the summary is higher freedom. Match degrees of freedom to fragility — a system-of-record write is a "narrow bridge" `(source: …agent-skills/best-practices)` [ANTHROPIC-OFFICIAL].
- **Verifiable intermediate:** produce `sor-update-plan.json`, validate field names against the live schema (connector) before writing — plan-validate-execute for high-stakes writes `(source: …agent-skills/best-practices)` [ANTHROPIC-OFFICIAL]. Connector returning a schema beats a copied schema that rots `(source: …writing-tools-for-agents)` [ANTHROPIC-OFFICIAL].
- **Human-in-the-loop:** show the proposed system-of-record diff, then write (Ask-before-acting).

### 7.6 Drafting stakeholder communications

- **Pattern:** evaluator–optimizer. Draft → evaluate against the comms rubric/voice guide → refine until it passes; "clear evaluation criteria + iterative refinement provides measurable value" `(source: …building-effective-agents)` [ANTHROPIC-OFFICIAL].
- **Skill `drafting-stakeholder-comms`:** voice/tone in `references/comms-style-guide.md`; 3–5 before/after examples covering escalation, status nudge, exec apology, expansion intro (examples > description for tone) `(source: …agent-skills/best-practices)` [ANTHROPIC-OFFICIAL].
- **Role + why:** "You are a power user writing to a frustrated key stakeholder; the goal is to retain trust, not to be defensive — explain the fix and the prevention" (role + motivation generalizes) `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Hard gate:** instruction-level rule "Never send a stakeholder email; only draft into `/outputs/` and stop for my review." Anthropic explicitly designs Cowork so "consequential decisions remain with the user" `(source: https://www.anthropic.com/product/claude-cowork)` [ANTHROPIC-OFFICIAL].

---

## 8. Iteration, evaluation, observability, and scope control

### 8.1 Build evaluations *before* writing the skill

Anthropic's evaluation-driven development for skills `(source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)` [ANTHROPIC-OFFICIAL]:

1. Run Claude on a representative task **without** the skill; document specific failures/missing context.
2. Create **at least three** evaluation scenarios that test those gaps (query + input files + expected behaviors).
3. Establish a baseline (no skill).
4. Write the **minimum** instructions that close the gaps and pass.
5. Iterate: re-run evals, compare to baseline, refine.

"This ensures your Skill solves real problems rather than documenting imagined ones." There is no built-in eval runner — keep a simple eval doc/JSON per skill as your source of truth `(source: …agent-skills/best-practices)` [ANTHROPIC-OFFICIAL]. This is the operational form of Anthropic's standing rule: "prompt engineering is a science… test your prompts and iterate often" `(source: https://www.anthropic.com/news/prompt-engineering-for-business-performance)` [ANTHROPIC-OFFICIAL].

### 8.2 The Claude-A / Claude-B iteration loop

Develop with one Claude instance ("A"), test with a fresh instance that has the skill loaded ("B"), observe B on **real tasks not test scenarios**, bring specific failures back to A `(source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)` [ANTHROPIC-OFFICIAL]. You can build skills *with Claude* by running the workflow normally then asking "package what we just did into a skill" — Cowork's built-in plugin/skill creator captures steps, templates, and source locations `(source: https://claude.com/resources/tutorials/customize-claude-cowork)` [ANTHROPIC-OFFICIAL].

Watch for these specific signals and act on them `(source: …agent-skills/best-practices)` [ANTHROPIC-OFFICIAL]:

- Unexpected file-read order → structure isn't intuitive.
- Missed references → links not prominent enough.
- Same file read repeatedly → that content probably belongs in `SKILL.md`.
- Bundled file never accessed → unnecessary or poorly signaled.
- Skill not triggering → fix the **description** (it's almost always the description).

### 8.3 Observability of what Cowork did

Cowork shows its plan before acting and you can steer at any step `(source: https://www.anthropic.com/product/claude-cowork)` `(source: https://anthropic.skilljar.com/introduction-to-claude-cowork)` [ANTHROPIC-OFFICIAL]. Build observability into the skills themselves:

- **Mandate a change log.** Have skills write `outputs/what-changed.md` listing every decision/edit, and **cite source filenames** in synthesized reports `(source: https://karozieminski.substack.com/p/claude-cowork-guide-plugins-memory-sub-agents-tips)` [COMMUNITY] — consistent with Anthropic's transparency principle `(source: https://www.anthropic.com/research/building-effective-agents)` [ANTHROPIC-OFFICIAL].
- **Run-status files** for long jobs (per §5/§6) double as an audit trail and a resume point.
- **Enterprise-level:** admins can track Cowork usage across the org and control spend `(source: https://claude.com/product/cowork)` `(source: https://thenewstack.io/anthropic-takes-claude-cowork-out-of-preview-and-straight-into-the-enterprise/)` [ANTHROPIC-OFFICIAL; COMMUNITY reporting].

### 8.4 Failure recovery & scope control

- **Dry-run on dummy/non-critical files** before processing real stakeholder data `(source: https://karozieminski.substack.com/p/claude-cowork-guide-plugins-memory-sub-agents-tips)` [COMMUNITY].
- **Three-question scope frame** at task kickoff: (1) What does "done" look like? (2) What context can't Claude guess? (3) What constraints apply? `(source: https://karozieminski.substack.com/p/claude-cowork-guide-plugins-memory-sub-agents-tips)` [COMMUNITY] — this operationalizes Anthropic's "specify task, intent, and constraints upfront in the first turn" guidance for maximizing autonomy without thrash `(source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Scope clamps in skills:** explicit "do only X; don't touch Y; don't add Z" — counters the model's overengineering tendency `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].
- **Least-folder-access + Ask-before-acting** for any workflow that can email stakeholders, post to Slack, or write to the system of record `(source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork)` [ANTHROPIC-OFFICIAL].
- **Reversibility clause** in instructions (§4.2) so the agent self-gates on destructive/visible actions `(source: …claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL].

---

## 9. Governance for a team rollout

### 9.1 Personal vs. shared, and the distribution path

- In **claude.ai/Cowork, custom Skills are per-user** with no central admin distribution; the supported team mechanism is **Plugins through admin-managed marketplaces** `(source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)` `(source: https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork)` [ANTHROPIC-OFFICIAL]. Architect the library as a **plugin** (skills + connectors + sub-agents) from the start if it's meant to be shared.
- **Org-managed plugins can't be edited by end users**, which gives you a controlled, consistent baseline; admins can also restrict which plugins install and disable local MCP servers `(source: https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork)` [ANTHROPIC-OFFICIAL].
- Build with `Plugin Create` (built-in) or fork an Anthropic role template (sales/operations/etc.) `(source: https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork)` [ANTHROPIC-OFFICIAL].

### 9.2 Change management & review process

- Treat the plugin like software: **eval suite per skill** (≥3 scenarios), reviewed before publish; iterate via the Claude-A/B loop and **incorporate teammate feedback** ("does it trigger when expected? what's missing?") to cover blind spots in your own usage `(source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)` [ANTHROPIC-OFFICIAL].
- Version skills with an "Old patterns" `<details>` section instead of dating instructions, so deprecations don't rot the doc `(source: …agent-skills/best-practices)` [ANTHROPIC-OFFICIAL].
- Enterprise GA added the governance layer: role-based access (group/SCIM-based), private plugin marketplaces, usage tracking, spend controls, and a curated connector set — design rollout around admin distribution, not individual uploads `(source: https://thenewstack.io/anthropic-takes-claude-cowork-out-of-preview-and-straight-into-the-enterprise/)` `(source: https://techcrunch.com/2026/02/24/anthropic-launches-new-push-for-enterprise-agents-with-plugins-for-finance-engineering-and-design/)` [COMMUNITY, reporting Anthropic announcements].

### 9.3 Secrets and sensitive stakeholder data

- **Treat installing a skill/plugin like installing software**: only from trusted sources; audit every bundled file (SKILL.md, scripts, resources) for unexpected network calls or off-purpose tool use. Skills that fetch external URLs are higher risk; a malicious skill can exfiltrate data or misuse tools `(source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)` [ANTHROPIC-OFFICIAL].
- **Don't bake secrets or PII into skills/instructions.** Skills are reusable, shareable artifacts and **not ZDR-eligible** (skill definitions/execution data follow standard retention) `(source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)` [ANTHROPIC-OFFICIAL]. Reference credentials via connectors/auth, not literal values; reference stakeholder data via connectors at runtime rather than copying it into a skill.
- **Least access by default:** connect only the folders and connectors a workflow needs; use Ask-before-acting for stakeholder-facing/system-of-record-writing flows; require human approval before sending external comms (Anthropic designs Cowork so consequential decisions stay with the user) `(source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork)` `(source: https://www.anthropic.com/product/claude-cowork)` [ANTHROPIC-OFFICIAL].
- **PII handling pattern:** Anthropic's own business example uses few-shot redaction with XML-tagged input/output pairs — encode redaction as a low-freedom skill step with examples for any workflow handling stakeholder PII `(source: https://www.anthropic.com/news/prompt-engineering-for-business-performance)` [ANTHROPIC-OFFICIAL].

### 9.4 Onboarding teammates

- Ship the plugin + a short "how we use Cowork on the team" doc covering: the project skeleton, which connectors to enable, approval-mode policy, and the secrets/PII rules above.
- Anthropic's official onboarding path is the Skilljar course "Introduction to Claude Cowork" (first launch → confident daily use, context shaping the plan, setting up plugins/skills, steering, safe operation, plugin validation, team sharing) — use it as the baseline ramp `(source: https://anthropic.skilljar.com/introduction-to-claude-cowork)` [ANTHROPIC-OFFICIAL].
- The skill-authoring mindset to teach: a skill is "an onboarding guide for a new hire" — concise, only context Claude doesn't already have, organized like a table of contents `(source: https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)` `(source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)` [ANTHROPIC-OFFICIAL].

---

## 10. Build checklist (use this every time)

- [ ] Started from connectors + tight instructions + one scoped folder, not a pile of skills `(customize-claude-cowork)` [ANTHROPIC-OFFICIAL]
- [ ] Each repeated workflow captured as a skill; one-offs left as prompts `(teach-claude-your-way…)` [ANTHROPIC-OFFICIAL]
- [ ] Live data via connectors; no copied project/schema lists `(effective-context-engineering)` [ANTHROPIC-OFFICIAL]
- [ ] `description` is third-person, specific, slightly pushy, with trigger terms `(agent-skills/best-practices)` [ANTHROPIC-OFFICIAL]
- [ ] `SKILL.md` < 500 lines; references one level deep; domain-split reference files `(agent-skills/best-practices)` [ANTHROPIC-OFFICIAL]
- [ ] Degrees of freedom matched to fragility (low for system-of-record writes/redaction) `(agent-skills/best-practices)` [ANTHROPIC-OFFICIAL]
- [ ] 3–5 XML-tagged examples for any classification/tone task `(claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL]
- [ ] Role + the *why* stated; positive instructions; explicit scope ("every section") `(claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL]
- [ ] Skill triggering not over-prompted (no "CRITICAL/MUST" shouting) `(claude-prompting-best-practices)` [ANTHROPIC-OFFICIAL]
- [ ] Self-check + change log + source citations built into the skill `(claude-prompting-best-practices; building-effective-agents)` [ANTHROPIC-OFFICIAL]
- [ ] Reversibility clause + Ask-before-acting on stakeholder-facing/system-of-record steps `(claude-prompting-best-practices; get-started-cowork)` [ANTHROPIC-OFFICIAL]
- [ ] ≥3 evals written before the skill; iterated via Claude-A/B on real tasks `(agent-skills/best-practices)` [ANTHROPIC-OFFICIAL]
- [ ] Shareable library packaged as a plugin for admin distribution; no secrets/PII baked in `(use-plugins-in-cowork; agent-skills/overview)` [ANTHROPIC-OFFICIAL]

---

## Sources

1. https://claude.com/product/cowork — [ANTHROPIC-OFFICIAL] Cowork product page; capabilities, computer use, plan-then-approve, controlled access, enterprise admin/spend tracking, plugins/connectors. Accessed 2026-05-18; reflects post-GA product.
2. https://www.anthropic.com/product/claude-cowork — [ANTHROPIC-OFFICIAL] Cowork overview; "human oversight," "consequential decisions remain with the user." Accessed 2026-05-18.
3. https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices — [ANTHROPIC-OFFICIAL] Single reference for prompt engineering (Opus/Sonnet/Haiku 4.x): clarity, examples, XML, roles, thinking/effort, tool use, agentic systems, autonomy/safety, literalism, over/under-triggering. Accessed 2026-05-18; covers Opus 4.7.
4. https://www.anthropic.com/research/building-effective-agents — [ANTHROPIC-OFFICIAL] Workflows vs agents, building blocks, the five workflow patterns, agent pattern, ACI design, simplicity/transparency/tool-docs principles. (Originally published Dec 2024; evergreen.)
5. https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents — [ANTHROPIC-OFFICIAL] Context engineering vs prompt engineering, context rot, right-altitude system prompts, bloated-toolset failure mode, just-in-time retrieval, compaction, structured note-taking, sub-agents. (2025.)
6. https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview — [ANTHROPIC-OFFICIAL] Skill structure, YAML frontmatter rules, three-level progressive disclosure, where skills work, sharing scope asymmetry, security, ZDR exclusion. Accessed 2026-05-18.
7. https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices — [ANTHROPIC-OFFICIAL] Concise-is-key, degrees of freedom, description writing, naming (gerund), progressive-disclosure patterns, one-level-deep references, workflows/checklists, feedback loops, evaluation-first, Claude-A/B iteration, anti-patterns, scripts, plan-validate-execute. Accessed 2026-05-18.
8. https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills — [ANTHROPIC-OFFICIAL] Skills design philosophy: progressive disclosure, composability, "onboarding guide for a new hire," skills complement MCP. (2025.)
9. https://www.anthropic.com/engineering/writing-tools-for-agents — [ANTHROPIC-OFFICIAL] Five tool-design principles (consolidate to workflows, namespacing, meaningful context, token efficiency, prompt-engineered descriptions) and prototype-evaluate-collaborate. (2025.)
10. https://www.anthropic.com/news/prompt-engineering-for-business-performance — [ANTHROPIC-OFFICIAL] Prompting as a science; step-by-step reasoning, few-shot with edge cases, prompt chaining; PII redaction with XML pairs; "intern on first day." Published 2024-02-29.
11. https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — [ANTHROPIC-OFFICIAL Help Center] Global/folder instructions, projects (files/context/instructions/memory), controlled file & network access, two approval modes, delete confirmation, `/schedule` (awake + app open). Accessed 2026-05-18.
12. https://claude.com/resources/tutorials/customize-claude-cowork — [ANTHROPIC-OFFICIAL] Three-level customization ladder (context/tools → skills → plugins), connectors, two-tier instructions, "package what we just did into a skill," sequential rollout. Accessed 2026-05-18.
13. https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork — [ANTHROPIC-OFFICIAL Help Center] Plugins bundle skills/connectors/sub-agents; install/customize; Plugin Create; admin marketplaces; org-managed plugins not editable; trust/restriction controls. Accessed 2026-05-18.
14. https://claude.com/resources/tutorials/teach-claude-your-way-of-working-using-skills — [ANTHROPIC-OFFICIAL] Skills vs instructions vs projects vs prompting; "skills work everywhere"; create by example; trigger by name/purpose. Accessed 2026-05-18 (301 from support.claude.com article 12580051).
15. https://anthropic.skilljar.com/introduction-to-claude-cowork — [ANTHROPIC-OFFICIAL Academy/Skilljar] Onboarding course: describe-plan-execute-steer, standing context/global instructions, skills/plugins setup, steering long tasks, safe operation, plugin validation, team sharing. Accessed 2026-05-18.
16. https://www.anthropic.com/webinars/future-of-ai-at-work-introducing-cowork — [ANTHROPIC-OFFICIAL] Cowork positioning ("does the work alongside you"), admin/team rollout guidance framing; recorded 2026-01-30.
17. https://thenewstack.io/anthropic-takes-claude-cowork-out-of-preview-and-straight-into-the-enterprise/ — [COMMUNITY, reporting Anthropic] GA = governance layer for CIOs; role-based access, private plugin marketplaces, usage/spend controls, connector list. ~2026-04 (GA reporting).
18. https://pasqualepillitteri.it/en/news/755/anthropic-managed-agents-cowork-ga-april-9-2026 — [COMMUNITY, reporting Anthropic] Cowork GA dated April 9, 2026.
19. https://techcrunch.com/2026/02/24/anthropic-launches-new-push-for-enterprise-agents-with-plugins-for-finance-engineering-and-design/ — [COMMUNITY, reporting Anthropic] Role-based plugin templates, admin private marketplaces, expanded connectors. Published 2026-02-24.
20. https://generativeprogrammer.com/p/skill-authoring-patterns-from-anthropics — [COMMUNITY, summarizing Anthropic best-practices] "Explain the why," descriptions slightly pushy due to under-trigger tendency, splitting files for token efficiency.
21. https://karozieminski.substack.com/p/claude-cowork-guide-plugins-memory-sub-agents-tips — [COMMUNITY] Tested power-user tips: skill/memory/connector/plugin boundaries, sub-agent parallelism numbers, project memory needs explicit files, change-log/source-citation observability, three-question scope frame, dry-run-on-dummy-files, folder discipline. Published ~2026-03.

---

## Confidence & gaps

**High confidence (multiple primary Anthropic sources):** prompt-engineering principles and current-model behaviors; the workflow-vs-agent framework and five patterns; context-engineering (context rot, just-in-time, compaction, sub-agents); the entire Skills architecture, frontmatter rules, progressive disclosure, authoring best practices, and evaluation/iteration loop; tool/connector design principles; Cowork's three-level customization ladder; Cowork projects, instructions tiers, approval modes, file-access model, scheduled-task constraints; skill/plugin sharing-scope asymmetry; security and ZDR status.

**Medium confidence:** Cowork **project memory mechanics** (per-project summary, ~24h auto-update "useful bits") — stated in the official Help Center as fetched, but the exact cadence/granularity could change; verify against the live Help Center before relying on precise behavior. The practical "no automatic cross-session memory outside projects" is consistent across official + community sources but treat the auto-summary as best-effort, not a guaranteed store of precise facts (the doc recommends explicit files for must-persist data).

**Lower confidence / community-sourced:** specific sub-agent speedup numbers (~30 min → ~4 min for 10 files) and the folder-structure/three-question conventions come from a single reputable practitioner; directionally aligned with Anthropic guidance but not Anthropic-verified figures. Enterprise GA specifics (marketplace controls, GA date of 2026-04-09) — *reconfirmed 2026-05-18* against the official `claude.com/blog/cowork-for-enterprise` post (GA on all paid plans, macOS + Windows, RBAC + group spend limits) and the recovered CNBC 2026-02-24 article (connectors Google Drive/Gmail/DocuSign/FactSet; financial-analysis/engineering/HR plugins); now official/high confidence rather than press-only.

**Known gaps:** *Resolved 2026-05-18 via the Claude Chrome extension* — the canonical official GA post is `claude.com/blog/cowork-for-enterprise` (verified live: "Cowork is now generally available on all paid plans," macOS + Windows, RBAC + group spend limits). There is no separate `anthropic.com/news/claude-cowork`; that 404 was a non-issue. GA details now rest on a verified official source, not product page + press alone. I did not find an Anthropic prompt-library entry specifically branded for this domain; the prompt patterns here are derived from Anthropic's general prompt-engineering and business-performance guidance applied to real-world workflows, not lifted from a domain-specific official template. Anthropic does not currently provide a built-in skill-evaluation runner — the eval discipline must be implemented by you.

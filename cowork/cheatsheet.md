# Claude Cowork — Power User Cheat-Sheet

*One-page field reference. Distilled from the sourced docs in this folder — citations live there; this page is the gist. Compiled 2026-05-18; re-check beta items against the live Anthropic help center.*

---

### Mental model

**Cowork = Claude Code's brain, in a desktop GUI, sandboxed, for non-coders.** Same agentic engine; the *behavior* transfers, the *cockpit* doesn't. Code runs in a real terminal/shell; Cowork runs code in an **isolated VM**, touches only **folders you grant**, and is steered through a GUI + Skills/Connectors instead of slash commands. Desktop-only (no web/CLI). **Not** a detached cloud agent — the app must stay open while a task runs.

### The 4 building blocks — don't conflate them

| Block | What it's for | Lifetime |
|---|---|---|
| **Instructions** | Always-on rules, tone, "never email a stakeholder without my OK" | Every task (global or project) |
| **Skill** | A repeatable *procedure* (the way we do X) — loads only when relevant | Persistent; on-demand |
| **Memory** | What Claude *learned* about a workstream/your prefs | **Project-scoped — gone in standalone sessions** |
| **Connector** | Live data/actions (system of record, Slack, Drive, Gmail) | Persistent auth; never copy a list, query it |

Rule: once → just prompt. Repeatedly → make a Skill. Fact that accumulates → Memory. Universal rule → Instructions. Live data → Connector.

### Writing a good Skill (the high-leverage rules)

- **The `description` is the most important thing you write** — it's how Claude decides to use the skill. Third person, says *what it does + when to use it*, includes trigger words, slightly "pushy."
  - ✅ `Prepares a review-deck brief: pulls usage/status data, drafts the deck outline. Use when the user asks to prep a recurring business review, build a business review, or get ready for a status review.`
  - ❌ `Helps with stakeholder meetings.`
- **Keep `SKILL.md` under ~500 lines.** Push detail (rubrics, templates, schemas) into reference files **one level deep**.
- **Name:** lowercase-hyphens, gerund style — `preparing-review-decks`, `triaging-requests`. Not `helper`/`misc-utils`.
- **Match freedom to risk:** prose steps where judgment varies; exact script ("do not modify") for fragile/destructive steps (system-of-record writes, redaction).
- **Explain the *why*, don't just dictate** — Claude generalizes from the reason to cases you didn't spell out. Avoid ALL-CAPS ALWAYS/NEVER.
- **Use scripts for anything deterministic** (dedupe, schema-validate) — more reliable, costs no context.
- **Build 3+ evals before writing the skill.** Iterate: one Claude writes it, a fresh Claude *uses* it on real tasks; fix what the description/structure caused.
- **No secrets/PII in skills.** Treat installing a skill/plugin like installing software — only trusted sources.

### Prompting the current models (Opus/Sonnet 4.x)

- They're **literal** — state scope ("apply to every section, not just the first").
- **3–5 examples in `<example>` tags** beat any description for tone/format.
- Give Claude a **role**; add the **why**; say what *to* do (not what not to).
- Be explicit **action vs. analysis**: "update the tracker" acts; "suggest changes" only suggests.
- **Don't over-prompt triggering** — no "CRITICAL: you MUST". If a skill under-fires, fix the *description*, don't shout.
- **Scope clamp** for deliverables: "only produce the requested brief; don't add files/sections."
- **Reversibility clause:** local/reversible actions freely; **ask before** anything external or hard to undo (send email, post Slack, write system of record, delete).

### "Where did my Claude Code slash command go?"

| Claude Code | In Cowork |
|---|---|
| `/init`, `/memory`, `CLAUDE.md` | Projects + Instructions (Customize sidebar) |
| `/mcp` | Connectors (Customize → Connectors) |
| `/permissions`, `/sandbox` | "Ask before acting" / "Act without asking" toggle |
| `/plan` | Approve Claude's plan before it acts |
| `/agents` | Sub-agents bundled inside plugins |
| `/plugin` | Customize → Browse plugins |
| `/schedule` | **Same** — `/schedule` or the Scheduled sidebar |
| custom `.claude/commands/*.md` | **Skills** — same `SKILL.md`, type `/` to see them |
| `/compact`, `/clear`, `/resume`, `/diff`, hooks, `settings.json` | **No equivalent** — CLI-only, by design |

The portable unit is the **Skill** (open standard — one `SKILL.md` works in Code *and* Cowork). Hooks/`settings.json` do **not** port.

### Workflow patterns (pick the shape per task)

| Pattern | Use it for |
|---|---|
| Prompt chaining | Notes → fields → system of record → confirm; draft → review → revise |
| Routing | Request triage (classify → specialized handling) |
| Parallelization | Summarize 40 requests / 25 transcripts at once |
| Orchestrator-workers | "Prep recurring business reviews for my whole portfolio" (worker per workstream) |
| Evaluator-optimizer | Draft a high-stakes stakeholder email → check vs. rubric → refine |

### Gotchas (set these expectations)

- **Memory is project-scoped** — standalone folder sessions start fresh. Use a **Project** for anything recurring; write must-keep facts to an explicit file.
- **App must stay open + awake** — closing it stops active *and* scheduled tasks. `/schedule` is for convenience, not mission-critical timing.
- **No session/result sharing** — hand off the produced file. Share team setups via **admin-managed plugins** (custom skills are otherwise per-user).
- **For IT/security:** code runs in a VM (protects the OS, **not your files**); Cowork history is **local per user**, outside standard retention, **not in the Compliance API**. Default to "Ask before acting."

### First week

1. Make a **Project** per project/program (folder + tight Instructions). Grant only the folder you need.
2. Connect only the connectors a workflow needs (system of record, ticketing, Slack/Gmail).
3. Run a real task by just describing it. When you repeat it, say "package what we just did into a skill."
4. Keep "Ask before acting" on for anything stakeholder-facing.
5. Promote your best skills into a shared **plugin** for the team (via admin).

---
*Full detail & sources: `overview-and-cowork-vs-code.md` · `skills.md` · `memory.md` · `workflow-architecture-and-prompting.md` · `platform-deep-dive.md` · `slash-commands-and-customization.md`*

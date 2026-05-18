# Claude Cowork — best-practices research

A technically-deep, **exhaustively source-linked** brief on Claude Cowork best practices — skills, memory, workflow architecture, prompt engineering, and how Cowork differs from Claude Code. Written for a **technically literate power user** who builds skills/workflows and does heavy prompt engineering for a team, and who is approaching Cowork with a Claude Code power-user's mental model. [best-practices.md](best-practices.md) adds the **practitioner delta** (best practices *not* in the upfront docs) through a **Customer Success Manager** usage lens.

Every non-obvious claim in every doc carries an inline source URL. Each doc ends with a numbered `## Sources` list and a `## Confidence & gaps` section separating Anthropic-official facts from community/press inference and flagging staleness.

## How it was made (methodology)

Compiled **2026-05-18** by six parallel Opus research agents, each owning one slice and instructed to verify against **live web research** (Anthropic knowledge is past training cutoff and Cowork is fast-moving), prioritize official Anthropic sources (`docs.claude.com` / `code.claude.com`, `support.claude.com` help center, `claude.com/blog`, `anthropic.com`, `privacy.claude.com`, Anthropic Academy), cite only pages actually fetched, and explicitly mark official vs. community and call out gaps.

Because Cowork shipped Jan 2026 → GA Apr 2026 with a rapid release cadence, **re-verify anything tagged beta/preview against the live Anthropic help center before stakeholder-facing commitments.** Where official docs and community reports disagree, the docs say so rather than papering over it.

A **second research wave (also 2026-05-18)** ran six more parallel Opus agents tasked specifically with the **practitioner delta** — best practices *not* in the upfront official documentation — and a **Customer Success Manager** usage lens. Its synthesis is [best-practices.md](best-practices.md). It leans more heavily on community/practitioner sources (clearly tiered and flagged) than the official-doc-anchored deep dives, because that delta is the point; treat its CSM workflow layer as directional craft, not Anthropic-validated.

## Source-verification pass (2026-05-18)

A second pass re-fetched, via the Claude Chrome extension, every source the agents had flagged as 404'd or access-blocked. Outcome:

- **CNBC, 2026-02-24** (`cnbc.com/.../anthropic-claude-cowork-office-worker.html`) — had returned **HTTP 403** to automated fetch; **fully recovered in-browser.** Confirms verbatim the Feb-24 facts the docs previously held only from a search summary: connectors *Google Drive, Gmail, DocuSign, FactSet*; plugins for *financial analysis, engineering, HR*; "transition into a true enterprise-grade product"; Kate Jensen / Peter McCrory on the record. Confidence on those facts upgraded.
- **Anthropic GA "404"** — non-issue. There is no `anthropic.com/news/claude-cowork`; the **canonical official GA post is `claude.com/blog/cowork-for-enterprise`** (claude.com is Anthropic's product domain), verified live: heading *"Making Claude Cowork ready for enterprise,"* *"Cowork is now generally available on all paid plans,"* macOS + Windows, role-based access controls + group spend limits. The docs already cite this URL.
- **GitHub `spec/agent-skills-spec.md`** — the repo file is now a one-line **stub redirecting to `agentskills.io/specification`**, which is exactly the canonical source the skills doc already used. Gap favorably closed.
- **The New Stack** (community/press) — re-fetch **denied at the browser permission prompt** (domain-specific). Left unverified; it was non-load-bearing, and the GA facts it backed are now independently confirmed by the official `claude.com/blog` post and the recovered CNBC article.

Each affected doc's `## Confidence & gaps` has been updated to reflect these resolutions.

## The documents

| Doc | What it covers | Read it for |
|---|---|---|
| [cheatsheet.md](cheatsheet.md) | One-page printable distillation of every doc below | Print-and-go quick reference |
| [best-practices.md](best-practices.md) | The **practitioner delta**, not the official basics: a **Customer Success Manager playbook** (knowledge-work-plugin fork path, connector reality, churn/QBR/VoC workflows), plus skill-eval/description-engineering, memory hygiene, the settled multi-agent answer, Opus 4.7 prompting deltas, the host-vs-VM/lethal-trifecta/compliance-gap depth, cost & adoption | "What do experienced users know that the docs don't say?" — and the CSM angle |
| [overview-and-cowork-vs-code.md](overview-and-cowork-vs-code.md) | What Cowork is; execution model; **full Code-vs-Cowork feature matrix**; rollout timeline | The big picture + the comparison table |
| [skills.md](skills.md) | Agent Skills deep dive: `SKILL.md` schema, progressive disclosure, authoring best practices, anti-patterns, evals, scope/distribution, Cowork specifics | Building good skills |
| [memory.md](memory.md) | Disambiguates the **four** different "memory" systems; which one Cowork actually uses; hygiene, governance, failure modes | Not confusing chat memory with project memory |
| [workflow-architecture-and-prompting.md](workflow-architecture-and-prompting.md) | Architecting a Cowork setup for a team: component map, prompt engineering for agents, workflow patterns, **concrete workflow blueprints** (recurring status reviews, review-deck prep, risk scoring, request triage, notes→system-of-record, comms), evals, governance | Designing the whole setup |
| [platform-deep-dive.md](platform-deep-dive.md) | The Cowork product surface: tasks/projects/scheduling, sandbox/VM, connectors, file handling, security/admin/governance, plans, limits, gotchas | Rollout, IT/security review |
| [slash-commands-and-customization.md](slash-commands-and-customization.md) | Why Code slash commands ≠ Cowork; the commands→Skills merge; **Code-construct → Cowork-equivalent mapping table** | The "my Claude Code slash commands aren't in Cowork" question |

## Two questions this answers up front

**"What features does Cowork have that Claude Code does/doesn't?"** → Same agentic engine (the Claude Agent SDK harness), different packaging. Full matrix in [overview-and-cowork-vs-code.md §5](overview-and-cowork-vs-code.md). Headlines:

- **Cowork only**, vs. Code: a desktop GUI, isolated-VM sandbox for all code/shell (Code runs in your real shell with your perms), opt-in per-folder file access, GUI permission modes, GUI plugin/connector browser, Dispatch mobile pairing, org RBAC/spend/analytics/OpenTelemetry governance.
- **Code only**, vs. Cowork: a terminal, the full `~70`-command slash palette, `settings.json` + **hooks**, local stdio MCP, deep Git/PR workflow, CLI flags/env vars, detached/background & web/cloud sessions.
- **Shared (different surface):** Skills, Memory, MCP (as "connectors"), subagents, plan-approval, `/schedule`, projects. Cowork is desktop-only (no web/CLI) and **not** a detached cloud agent — the app must stay open.

**"Why are there built-in Claude Code slash commands that don't appear in Cowork?"** → Most Code slash commands are **CLI session/repo/config plumbing** (`/init`, `/compact`, `/clear`, `/model`, `/permissions`, `/hooks`, `/review`, `/diff`, `/resume`, `/terminal-setup`…) hard-coded into the CLI; they have **no referent** in a managed desktop product, so they don't exist in Cowork — by design, not omission. Cowork *does* have a `/` menu, but it lists **Skills and installed plugin slash commands**, not CLI built-ins; most of those commands' *functions* moved into the GUI (Projects, Connectors, permission toggles, Customize sidebar). `/schedule` is one of the few genuinely shared. The portable customization layer is **Skills + plugins** (the open Agent Skills standard) — a `SKILL.md` authored once works in Code *and* Cowork. Hooks/`settings.json` do **not** port. Full detail + mapping table in [slash-commands-and-customization.md](slash-commands-and-customization.md).

## One-screen mental model

- **Cowork = Claude Code's brain, in a desktop GUI, sandboxed, for non-coders.** Behavior transfers; the cockpit doesn't.
- **Stop hunting for slash commands. Build Skills.** A Skill is the reusable, portable workflow unit. Description quality is the single highest-leverage thing you write (it's how Claude decides to use the skill). Keep `SKILL.md` < 500 lines; push detail to one-level-deep reference files; ship deterministic steps as scripts.
- **Four things, don't conflate them:** *Instructions* = always-on rules/tone · *Skills* = repeatable procedures (load on demand) · *Memory* = what Claude learned (project-scoped; **gone in standalone sessions**) · *Connectors* = live data, never a copied list.
- **Build evals before the skill.** Iterate with one Claude writing it and a fresh Claude using it on real tasks.
- **Govern it:** least-folder access, "Ask before acting" for anything stakeholder-facing/system-of-record-writing, no secrets/PII baked into skills, distribute team setups as **admin-managed plugins** (custom skills are otherwise per-user). Note for IT: Cowork history is **local per user**, outside standard retention, and **not in the Compliance API**.
- **For practitioners (and CSMs specifically): start from [best-practices.md](best-practices.md).** Don't build CS workflows from scratch — fork Anthropic's `knowledge-work-plugins` (`customer-support` + `sales`); the win is whole-corpus batch synthesis, **not** live CRM write-back (no native Gainsight/Catalyst/Planhat/Salesforce connector today); skill descriptions route on keywords, so engineer them against an eval set; one orchestrator owns context, sub-agents are read-only; treat every run as needing human review.

## Caveats

Researched 2026-05-18. Cowork is on a monthly release cadence; "beta/preview" items (computer use, Dispatch) and exact plan/governance scoping change fastest. Treat community/press figures (usage limits, connector counts) as directional only. Each doc's `## Confidence & gaps` is the authoritative statement of what is solid vs. inferred.

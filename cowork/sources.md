# Sources

Every external link used as a source across this brief, consolidated. Each
doc also carries its own inline citations and a numbered `## Sources` list;
this file is the union, deduplicated and grouped by trust tier.

Compiled **2026-05-18**. "Cited in" uses short doc names: `overview` =
`overview-and-cowork-vs-code.md`, `skills` = `skills.md`, `memory` =
`memory.md`, `workflow` = `workflow-architecture-and-prompting.md`,
`platform` = `platform-deep-dive.md`, `slash` =
`slash-commands-and-customization.md`, `bestprac` = `best-practices.md`.
`README.md` and `cheatsheet.md` distill the others and carry no independent
citations.

`best-practices.md` (a 2026-05-18 second research wave on the practitioner
delta + CSM lens) re-cites many entries already listed in the trust-tiered
union (sections 1–8 — notably the official Anthropic engineering/help-center
pages and the `knowledge-work-plugins` repo) and adds the new, mostly
community/practitioner sources consolidated in the **leading section directly
below**. Its own inline citations and `## Confidence & gaps` are authoritative
for that doc.

**Trust order**, applied throughout: Anthropic/Claude official (product,
blog, docs, help center, privacy, engineering/research) > the Agent Skills
open standard > Anthropic learning/webinar > reputable press > independent
community guides. Where official docs and community/press disagreed, the
docs say so rather than reconciling silently.

## Verification status (2026-05-18 pass)

A second pass re-fetched, via the Claude Chrome extension, every source the
research agents had flagged as 404'd or access-blocked:

- `cnbc.com/.../anthropic-claude-cowork-office-worker.html` — returned **HTTP
  403** to automated fetch; **fully recovered in-browser**. Confirms verbatim
  the 2026-02-24 connector/plugin facts.
- `claude.com/blog/cowork-for-enterprise` — the canonical official GA post
  (there is no `anthropic.com/news/claude-cowork`); **verified live**.
- `agentskills.io/specification` — the GitHub `spec/agent-skills-spec.md`
  file is now a stub redirecting here; **canonical, gap closed**.
- `thenewstack.io/...` — re-fetch **denied at the browser permission prompt**
  (domain-specific). Left unverified; non-load-bearing — the GA facts it
  backed are independently confirmed by the official `claude.com/blog` post
  and the recovered CNBC article.

---

## `best-practices.md` additions (2026-05-18 second wave — practitioner delta + CSM lens)

Moved to the top for visibility. Net-new sources not already listed in the
trust-tiered union (sections 1–8 below). Heavily community/practitioner-tiered
by design (the brief's value is the non-official delta); each is trust-tagged
and gap-flagged inline in `bestprac`. Re-cited official entries appear in the
tiered sections below.

**Official / Anthropic press / vendor-engineering**

- **knowledge-work-plugins — customer-support & sales SKILL.md / CONNECTORS.md** — https://github.com/anthropics/knowledge-work-plugins — fork-able CS/sales skills; vendor-agnostic connector placeholders. *Cited in: bestprac (also slash, workflow).*
- **Skill-creator — test, measure, refine** — https://claude.com/blog/improving-skill-creator-test-measure-and-refine-agent-skills — evals as model-upgrade tripwire, blind A/B. *Cited in: bestprac.*
- **Multi-agent research system (engineering)** — https://www.anthropic.com/engineering/multi-agent-research-system — 4×/15× token economics, fan-out rubric, delegation contract. *Cited in: bestprac.*
- **Models migration guide** — https://platform.claude.com/docs/en/about-claude/models/migration-guide — Opus 4.7 / Sonnet 4.6 behavioral deltas, new tokenizer. *Cited in: bestprac.*
- **Cowork desktop architecture overview** — https://support.claude.com/en/articles/14479288-claude-cowork-desktop-architecture-overview — host-vs-VM split, VM opaque to EDR. *Cited in: bestprac.*
- **Monitor Cowork with OpenTelemetry** — https://support.claude.com/en/articles/14477985-monitor-claude-cowork-activity-with-opentelemetry — Cowork OTel events, `prompt.id`, opt-in. *Cited in: bestprac.*
- **Schedule recurring tasks** — https://support.claude.com/en/articles/13854387-schedule-recurring-tasks-in-claude-cowork — fresh-session-per-run, no inter-run memory. *Cited in: bestprac (also platform).*
- **Cookbook — context-engineering tools** — https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools — store/query/encode decision framework. *Cited in: bestprac.*
- **Perplexity — designing/refining/maintaining Agent Skills** — https://research.perplexity.ai/articles/designing-refining-and-maintaining-agent-skills-at-perplexity — ≤50-word "Load when" descriptions, tax test, Gotchas Flywheel (press/vendor-eng). *Cited in: bestprac.*

**Community / practitioner**

- **GTM AI Podcast — Claude (Code/Cowork) for Customer Success** — https://www.gtmaipodcast.com/p/claude-code-for-customer-success — whole-corpus churn model, roster-seeded QBR (paywalled past intro). *Cited in: bestprac.*
- **Databar — Cowork CS churn prevention** — https://databar.ai/blog/article/claude-code-customer-success-churn-prevention — 60/40 composite score, champion-departure predictor. *Cited in: bestprac.*
- **Vitally — AI prompts for CS** — https://www.vitally.io/post/ai-prompts-for-cs — reusable QBR/VoC prompts (tool-agnostic). *Cited in: bestprac.*
- **tldv — Claude Cowork for CS** — https://tldv.io/blog/claude-cowork/ — least-privilege, no native CRM API, anti-hallucination gate. *Cited in: bestprac.*
- **eesel — Cowork Salesforce integration** — https://www.eesel.ai/blog/claude-cowork-salesforce-integration — no turnkey Salesforce connector. *Cited in: bestprac.*
- **Composio — Salesforce toolkit for Cowork** — https://composio.dev/toolkits/salesforce/framework/claude-cowork — 3rd-party MCP bridge. *Cited in: bestprac.*
- **Kent Broadbent — Cowork ↔ Google Workspace via gws** — https://kent-broadbent.medium.com/connecting-claudes-cowork-to-google-workspace-using-gws-f5057df35eb6 — native Google connectors read-only. *Cited in: bestprac.*
- **Product Compass — Cowork guide** — https://www.productcompass.pm/p/claude-cowork-guide — Ask/Allow permission discipline. *Cited in: bestprac.*
- **Scott Spence — measuring skill activation with sandboxed evals** — https://scottspence.com/posts/measuring-claude-code-skill-activation-with-sandboxed-evals — keyword-ish (not semantic) activation. *Cited in: bestprac.*
- **mager.co — the eval loop** — https://www.mager.co/blog/2026-03-08-claude-code-eval-loop/ — dual-gate harness, runs-per-query=3. *Cited in: bestprac.*
- **obra/superpowers — writing-skills best practices** — https://github.com/obra/superpowers/blob/main/skills/writing-skills/anthropic-best-practices.md — third-person description rule, one-level-deep references. *Cited in: bestprac.*
- **skillsmith.app — agent skill framework** — https://www.skillsmith.app/blog/agent-skill-framework — skill/sub-agent/sub-skill/MCP decision matrix. *Cited in: bestprac.*
- **Cognition — Multi-Agents: what's actually working** — https://cognition.ai/blog/multi-agents-working — 2026-04-22 reversal; single-threaded writes, read-only sub-agents. *Cited in: bestprac.*
- **Cognition — Don't build multi-agents** — https://cognition.ai/blog/dont-build-multi-agents — share-context / actions-carry-decisions. *Cited in: bestprac.*
- **Nicholas Rhodes — Cowork projects as business OS** — https://nicholasrhodes.substack.com/p/claude-cowork-projects-business-os — per-account vs per-function project structure. *Cited in: bestprac.*
- **dbreunig — how to fix your context** — https://www.dbreunig.com/2025/06/26/how-to-fix-your-context.html — poisoning/distraction/clash remediations. *Cited in: bestprac.*
- **claudefa.st — Auto Dream** — https://claudefa.st/blog/guide/mechanics/auto-dream — auto memory consolidation (Code lineage; Cowork parity unconfirmed). *Cited in: bestprac.*
- **Karo Zieminski — Notion connector as persistent memory** — https://karozieminski.substack.com/p/claude-cowork-notion-connector-persistent-memory-tips — connector-as-memory (paywalled). *Cited in: bestprac.*
- **Ruben Hassid — Cowork project + 21 cost hacks** — https://ruben.substack.com/p/claude-cowork-project — folder-as-bus handoff, PDF→md token lever. *Cited in: bestprac.*
- **PromptArmor — Cowork exfiltrates files** — https://www.promptarmor.com/resources/claude-cowork-exfiltrates-files — file-exfiltration-via-injection disclosure dated 2026-05-18 (unpatched as written). *Cited in: bestprac.*
- **Simon Willison — lethal trifecta (3 posts)** — https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/ , /2025/Sep/26/how-to-stop-ais-lethal-trifecta/ , /2025/Nov/2/new-prompt-injection-papers/ — lethal trifecta, cut-a-leg, Rule of Two. *Cited in: bestprac.*
- **repello.ai — Cowork security** — https://repello.ai/blog/claude-cowork-security — connector blast radius, RBAC scope limits, detections. *Cited in: bestprac.*
- **Harmonic Security — securing Cowork** — https://www.harmonic.security/resources/securing-claude-cowork-a-security-practitioners-guide — compliance-gap eDiscovery impact, endpoint-as-data-layer, MDM. *Cited in: bestprac.*
- **Elastic Security Labs — Cowork OTel monitoring** — https://www.elastic.co/security-labs/claude-code-cowork-monitoring-otel-elastic — OTel event types, SIEM detections. *Cited in: bestprac.*
- **GitHub anthropics/claude-code issues #36131, #44128** — https://github.com/anthropics/claude-code/issues/36131 — scheduled-task focus bug, queue-and-fire, one catch-up. *Cited in: bestprac.*
- **buildtolaunch — scheduled tasks vs routines vs loop; onboarding guide** — https://buildtolaunch.substack.com/p/claude-cowork-scheduled-tasks-vs-routines-vs-loop — failure-cost task routing, champion model, Day-Two problem. *Cited in: bestprac.*
- **FindSkill — computer-use/Cowork setup** — https://findskill.ai/blog/claude-computer-use-cowork-setup/ — long-session/Dispatch failure-rate estimates (single-source), Windows disk trap. *Cited in: bestprac.*
- **Hacker News thread (id=46612919)** — https://news.ycombinator.com/item?id=46612919 — shared budget pool, Cowork-vs-Code reliability/cost. *Cited in: bestprac.*

---

## 1. Anthropic / Claude — official product, blog & announcements

- **Claude Cowork product page** — https://www.anthropic.com/product/claude-cowork — what Cowork is, positioning, execution model. *Cited in: overview, platform, workflow, bestprac.*
- **Claude Cowork product page (claude.com)** — https://claude.com/product/cowork — product surface, plans. *Cited in: platform, workflow.*
- **Claude Enterprise** — https://www.anthropic.com/product/enterprise — enterprise plan scope, admin/governance. *Cited in: platform.*
- **Cowork research preview announcement** — https://claude.com/blog/cowork-research-preview — original Jan-2026 launch framing. *Cited in: overview, platform.*
- **Making Claude Cowork ready for enterprise (canonical GA post)** — https://claude.com/blog/cowork-for-enterprise — GA on all paid plans, macOS + Windows, RBAC, group spend limits. *Cited in: platform.* **Verified live in-browser 2026-05-18.**
- **Cowork plugins** — https://claude.com/blog/cowork-plugins — plugins in Cowork, distribution. *Cited in: slash.*
- **Claude Skills announcement** — https://claude.com/blog/skills — Skills launch and rationale. *Cited in: skills.*
- **Organization skills & the directory** — https://claude.com/blog/organization-skills-and-directory — org-managed skills, the directory. *Cited in: skills.*
- **Claude memory announcement** — https://claude.com/blog/memory — chat memory / synthesis system. *Cited in: memory.*
- **Create files in Claude** — https://claude.com/blog/create-files — file creation/handling. *Cited in: platform.*
- **Building agents with the Claude Agent SDK** — https://claude.com/blog/building-agents-with-the-claude-agent-sdk — the shared engine behind Code and Cowork. *Cited in: overview.*
- **Tutorial: customize Claude Cowork** — https://claude.com/resources/tutorials/customize-claude-cowork — customization surface, Skills/plugins. *Cited in: slash, workflow.*
- **Tutorial: teach Claude your way of working using Skills** — https://claude.com/resources/tutorials/teach-claude-your-way-of-working-using-skills — skill-authoring workflow. *Cited in: skills, workflow.*
- **Claude for small business** — https://www.anthropic.com/news/claude-for-small-business — plan tiers / SMB framing. *Cited in: overview, bestprac.*
- **Introducing Anthropic Labs** — https://www.anthropic.com/news/introducing-anthropic-labs — context for product line. *Cited in: platform.*
- **Anthropic Series G funding** — https://www.anthropic.com/news/anthropic-raises-30-billion-series-g-funding-380-billion-post-money-valuation — company scale/context. *Cited in: platform.*
- **Prompt engineering for business performance** — https://www.anthropic.com/news/prompt-engineering-for-business-performance — business-facing prompting guidance. *Cited in: workflow.*

## 2. Anthropic engineering & research

- **Effective context engineering for AI agents** — https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents — context window, memory, progressive disclosure. *Cited in: memory, workflow, bestprac.*
- **Equipping agents for the real world with Agent Skills** — https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills — Skills design philosophy. *Cited in: skills, workflow, bestprac.*
- **Writing tools for agents** — https://www.anthropic.com/engineering/writing-tools-for-agents — tool/skill interface design. *Cited in: workflow.*
- **Claude Code sandboxing** — https://www.anthropic.com/engineering/claude-code-sandboxing — sandbox/VM execution model. *Cited in: platform.*
- **Building effective agents** — https://www.anthropic.com/research/building-effective-agents — agent architecture patterns. *Cited in: workflow.*
- **Multi-agent research system** — https://www.anthropic.com/engineering/multi-agent-research-system — token economics, fan-out rubric, delegation contract, eval discipline. *Cited in: bestprac.*

## 3. Official documentation (developer & Claude Code)

- **Agent Skills — overview** — https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview — `SKILL.md` model, progressive disclosure. *Cited in: skills, memory, workflow.*
- **Agent Skills — best practices** — https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices — authoring best practices, anti-patterns. *Cited in: skills, workflow.*
- **Memory tool (`memory_20250818`)** — https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool — the API memory tool. *Cited in: memory, bestprac.*
- **Context editing** — https://platform.claude.com/docs/en/build-with-claude/context-editing — context management primitives. *Cited in: memory.*
- **Cookbook — context-engineering tools** — https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools — store/query/encode decision framework. *Cited in: bestprac.*
- **Claude prompting best practices** — https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices — canonical prompt-engineering guidance; reversibility clause. *Cited in: workflow, bestprac.*
- **Models migration guide** — https://platform.claude.com/docs/en/about-claude/models/migration-guide — Opus 4.7 / Sonnet 4.6 behavioral deltas, new tokenizer. *Cited in: bestprac.*
- **Claude Code — slash commands** — https://code.claude.com/docs/en/commands — the CLI command palette. *Cited in: overview, slash.*
- **Claude Code — Skills** — https://code.claude.com/docs/en/skills — Skills in Code. *Cited in: overview, skills, slash.*
- **Claude Code — plugins** — https://code.claude.com/docs/en/plugins — plugin system, distribution. *Cited in: slash.*
- **Claude Code — memory** — https://code.claude.com/docs/en/memory — `CLAUDE.md` / auto-memory. *Cited in: memory.*

## 4. Claude Help Center & privacy

- **Get started with Claude Cowork** — https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — foundational Cowork behavior. *Cited in: overview, skills, memory, platform, slash, workflow.*
- **Use Claude Cowork on Team & Enterprise plans** — https://support.claude.com/en/articles/13455879-use-claude-cowork-on-team-and-enterprise-plans — plan-tier scoping; local history / Compliance-API exclusion. *Cited in: overview, memory, bestprac.*
- **Use plugins in Claude Cowork** — https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork — plugin install/distribution in Cowork. *Cited in: overview, platform, slash, workflow.*
- **Use Claude Cowork safely** — https://support.claude.com/en/articles/13364135-use-claude-cowork-safely — safety/permissions model; ~1% injection rate. *Cited in: platform, bestprac.*
- **Let Claude use your computer in Cowork** — https://support.claude.com/en/articles/14128542-let-claude-use-your-computer-in-cowork — computer-use (beta). *Cited in: platform.*
- **Schedule recurring tasks in Claude Cowork** — https://support.claude.com/en/articles/13854387-schedule-recurring-tasks-in-claude-cowork — `/schedule` / recurring tasks. *Cited in: platform, bestprac.*
- **Assign tasks to Claude from anywhere in Cowork** — https://support.claude.com/en/articles/13947068-assign-tasks-to-claude-from-anywhere-in-cowork — Dispatch / task entry points. *Cited in: platform.*
- **Organize your tasks with projects in Claude Cowork** — https://support.claude.com/en/articles/14116274-organize-your-tasks-with-projects-in-claude-cowork — Projects + project memory. *Cited in: memory, bestprac.*
- **What are Skills?** — https://support.claude.com/en/articles/12512176-what-are-skills — Skills, end-user framing. *Cited in: skills, slash.*
- **Use Skills in Claude** — https://support.claude.com/en/articles/12512180-use-skills-in-claude — invoking skills. *Cited in: overview, skills, slash.*
- **How to create custom skills** — https://support.claude.com/en/articles/12512198-how-to-create-custom-skills — authoring custom skills. *Cited in: skills.*
- **Create a skill with Claude through conversation** — https://support.claude.com/en/articles/12599426-how-to-create-a-skill-with-claude-through-conversation — conversational skill-creator. *Cited in: slash.*
- **Provision & manage skills for your organization** — https://support.claude.com/en/articles/13119606-provision-and-manage-skills-for-your-organization — org skill governance. *Cited in: skills.*
- **Browse skills, connectors & plugins in one directory** — https://support.claude.com/en/articles/14328846-browse-skills-connectors-and-plugins-in-one-directory — the directory UI. *Cited in: slash.*
- **Use connectors to extend Claude's capabilities** — https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities — MCP-as-connectors. *Cited in: overview, platform.*
- **Use interactive connectors in Claude** — https://support.claude.com/en/articles/13454812-use-interactive-connectors-in-claude — interactive connector behavior. *Cited in: platform.*
- **Use Google Workspace connectors** — https://support.claude.com/en/articles/10166901-use-google-workspace-connectors — Drive/Gmail/Calendar connectors; OAuth send-scope over-grant. *Cited in: overview, platform, bestprac.*
- **Connectors collection** — https://support.claude.com/en/collections/15399129-connectors — connector index. *Cited in: platform.*
- **Use chat search & memory to build on previous context** — https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context — chat memory behavior. *Cited in: memory.*
- **Monitor Claude Cowork activity with OpenTelemetry** — https://support.claude.com/en/articles/14477985-monitor-claude-cowork-activity-with-opentelemetry — Cowork OTel events, `prompt.id`, opt-in. *Cited in: bestprac.*
- **Claude Cowork desktop architecture overview** — https://support.claude.com/en/articles/14479288-claude-cowork-desktop-architecture-overview — host-vs-VM split, VM opaque to EDR. *Cited in: bestprac.*
- **Release notes** — https://support.claude.com/en/articles/12138966-release-notes — version/feature cadence; re-check for beta items. *Cited in: platform.*
- **Privacy center** — https://privacy.claude.com — data handling root. *Cited in: memory.*
- **Is my data used for model training?** — https://privacy.claude.com/en/articles/7996868-is-my-data-used-for-model-training — training-data policy. *Cited in: memory.*
- **How long do you store my data?** — https://privacy.claude.com/en/articles/10023548-how-long-do-you-store-my-data — retention; Cowork history is local/outside standard retention. *Cited in: platform.*
- **How long do you store my organization's data?** — https://privacy.claude.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data — commercial ~30-day deletion, no default training. *Cited in: bestprac.*

## 5. Agent Skills open standard & Anthropic GitHub

- **Agent Skills standard — home** — https://agentskills.io/home — the open standard. *Cited in: skills.*
- **Agent Skills — specification** — https://agentskills.io/specification — canonical `SKILL.md` spec. *Cited in: skills.* **Canonical target of the now-stubbed GitHub spec file.**
- **anthropics/skills — README** — https://github.com/anthropics/skills/blob/main/README.md — reference skill repo. *Cited in: skills.*
- **anthropics/skills — skill-creator SKILL.md** — https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md — worked authoring example; "pushy" descriptions, eval loop. *Cited in: skills, bestprac.*
- **anthropics/knowledge-work-plugins** — https://github.com/anthropics/knowledge-work-plugins — official knowledge-work plugin pack (customer-support & sales SKILL.md / CONNECTORS.md). *Cited in: slash, bestprac.*
- **Skill-creator — test, measure, refine (blog)** — https://claude.com/blog/improving-skill-creator-test-measure-and-refine-agent-skills — evals as model-upgrade tripwire, blind A/B. *Cited in: bestprac.*

## 6. Anthropic learning / webinar

- **Anthropic Academy — Introduction to Claude Cowork** — https://anthropic.skilljar.com/introduction-to-claude-cowork — official onboarding course. *Cited in: workflow.*
- **Webinar: Future of AI at Work — introducing Cowork** — https://www.anthropic.com/webinars/future-of-ai-at-work-introducing-cowork — official webinar. *Cited in: workflow.*

## 7. Press & news coverage

- **TechCrunch, 2026-01-12 — "Claude Code without the code"** — https://techcrunch.com/2026/01/12/anthropics-new-cowork-tool-offers-claude-code-without-the-code/ — launch coverage. *Cited in: overview.*
- **TechCrunch, 2026-02-24 — enterprise agents push** — https://techcrunch.com/2026/02/24/anthropic-launches-new-push-for-enterprise-agents-with-plugins-for-finance-engineering-and-design/ — Feb-24 plugins/connectors. *Cited in: workflow.*
- **CNBC, 2026-02-24** — https://www.cnbc.com/2026/02/24/anthropic-claude-cowork-office-worker.html — Feb-24 connectors (Drive, Gmail, DocuSign, FactSet) & plugins (finance, engineering, HR). *Cited in: overview.* **Was HTTP 403; recovered in-browser 2026-05-18 — facts confirmed verbatim.**
- **9to5Mac, 2026-04-09 — enterprise features & managed agents** — https://9to5mac.com/2026/04/09/anthropic-scales-up-with-enterprise-features-for-claude-cowork-and-managed-agents/ — Apr-09 enterprise scale-up. *Cited in: overview.*
- **TestingCatalog — Cowork GA** — https://www.testingcatalog.com/anthropic-launches-claude-cowork-in-general-availability/ — GA coverage. *Cited in: overview.*
- **The New Stack — Cowork out of preview** — https://thenewstack.io/anthropic-takes-claude-cowork-out-of-preview-and-straight-into-the-enterprise/ — GA/enterprise coverage. *Cited in: workflow.* **Re-fetch denied at browser permission prompt; unverified, non-load-bearing.**
- **CybersecurityNews — Projects feature** — https://cybersecuritynews.com/projects-feature-claude-cowork-desktop/ — Projects coverage. *Cited in: overview.*
- **Pasquale Pillitteri — managed agents / Cowork GA, 2026-04-09** — https://pasqualepillitteri.it/en/news/755/anthropic-managed-agents-cowork-ga-april-9-2026 — GA timeline. *Cited in: workflow.*

## 8. Independent guides, tutorials & community blogs

Directional/community — used for synthesis and cross-checking, never as the
sole basis for an Anthropic-official claim. (Sources unique to
`best-practices.md` are in the leading section at the top of this file.)

- **DataCamp — Claude Cowork tutorial** — https://www.datacamp.com/tutorial/claude-cowork-tutorial — hands-on walkthrough. *Cited in: overview.*
- **Matt Geer — difference between Claude.ai, Cowork & Code** — https://mattgeer.com/difference-between-claude-ai-cowork-and-code/ — product comparison. *Cited in: overview.*
- **Karo Zieminski — Cowork guide: plugins, memory, sub-agents, tips** — https://karozieminski.substack.com/p/claude-cowork-guide-plugins-memory-sub-agents-tips — practitioner guide. *Cited in: memory, workflow.*
- **Ryan & Matt Data Science — Claude Cowork projects** — https://ryanandmattdatascience.com/claude-cowork-projects/ — projects/memory walkthrough. *Cited in: memory, bestprac.*
- **Good AI Guide — project memory in Claude** — https://www.goodaiguide.com/guides/project-memory-claude/ — project-memory mechanics. *Cited in: memory.*
- **Simon Willison — Claude memory (2025-09-12)** — https://simonwillison.net/2025/Sep/12/claude-memory/ — independent analysis of chat memory. *Cited in: memory.*
- **Generative Programmer — skill-authoring patterns from Anthropic's** — https://generativeprogrammer.com/p/skill-authoring-patterns-from-anthropics — skill-authoring patterns. *Cited in: workflow.*
- **Sherlock — how to write skills for Claude Code & Cowork** — https://sherlock.xyz/post/how-to-write-skills-for-claude-code-and-cowork — skill-authoring guide. *Cited in: skills.*
- **MindStudio — Claude Code skills vs slash commands** — https://www.mindstudio.ai/blog/claude-code-skills-vs-slash-commands-2 — skills-vs-commands framing. *Cited in: skills.*
- **Morph — Claude Code plugins vs skills** — https://www.morphllm.com/claude-code-plugins-vs-skills — plugins-vs-skills framing. *Cited in: skills.*
- **APIYI — subscription/Agent SDK billing split (2026-06)** — https://help.apiyi.com/en/anthropic-claude-subscription-agent-sdk-billing-split-june-2026-en.html — billing-split note (directional, third-party). *Cited in: overview.*

---

*All links collected from the documents in this directory as of 2026-05-18.
Cowork is on a monthly release cadence — re-verify beta/preview-tagged facts
against the live Anthropic help center before stakeholder-facing commitments.
Each doc's `## Confidence & gaps` is the authoritative statement of what is
solid vs. inferred; `best-practices.md` leans community-tiered by design.*

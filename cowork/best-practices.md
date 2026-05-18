# Claude Cowork — Best Practices (the practitioner delta, CSM lens)

The stuff **not in the upfront docs**: battle-tested skill/memory/architecture
patterns, plus how a **Customer Success Manager** gets real leverage.
Written for a technically literate reader. The official-doc basics live in the
other docs in this folder (`skills.md`, `memory.md`,
`workflow-architecture-and-prompting.md`, `platform-deep-dive.md`); this doc is
the **delta** and does not restate them.

**Researched 2026-05-18** by a second wave of parallel Opus agents doing live
web research. Cowork ships ~monthly and several load-bearing items below are
beta or community-sourced — every claim carries an inline source; trust tier is
marked (**official** / **press** / **community**). Re-verify beta/community
items before stakeholder commitments. `## Confidence & gaps` at the end is
authoritative on what's solid.

---

## 0. The one thing to internalize first

Cowork is a **"research preview"-grade agent in a desktop GUI** — the caveat is
load-bearing, not boilerplate. Community measurement puts long-session and
scheduled-Dispatch failure rates high enough that **every run needs human
review; it is delegation, not automation** ([findskill.ai](https://findskill.ai/blog/claude-computer-use-cowork-setup/), community — single-source estimate, see gaps). Anthropic's own framing: *"consequential decisions remain with the user"* ([anthropic.com/product/claude-cowork](https://www.anthropic.com/product/claude-cowork), official). Design every CS workflow so the **natural pause point coincides with an approval gate**, keep **"Ask before acting"** on for anything that emails a customer or writes a system of record, and treat the produced *file* — not the session — as the deliverable.

Mental model in three lines:

1. **Cowork turns CS *exports* into CS *artifacts*.** The leverage is batch synthesis over a whole NPS dump / renewal pipeline in one ~1M-token context — **not** live CRM write-back (no native Gainsight/Catalyst/Planhat/Salesforce connector today; see §2).
2. **Skill = portable procedure; Project = durable knowledge; Memory = learned facts (project-scoped, gone in standalone sessions); Connector = live data.** Don't conflate them — that confusion is the #1 cause of "Claude forgot."
3. **Cowork is invisible to enterprise governance** (excluded from Audit Logs, Compliance API, Data Exports — local-only history). For a CS org holding customer PII/financials this is a hard constraint, not a tuning knob.

---

## 1. The CSM playbook

### 1.1 Don't build from scratch — fork Anthropic's knowledge-work plugins

There is **no dedicated "customer-success" plugin**, but Anthropic open-sourced 11 plugins; two are directly load-bearing ([github.com/anthropics/knowledge-work-plugins](https://github.com/anthropics/knowledge-work-plugins), official). They're **markdown only — fork the repo, edit, done; no build/infra** ([README](https://github.com/anthropics/knowledge-work-plugins/blob/main/README.md), official). Skills are vendor-agnostic by design: connectors are category placeholders (`~~CRM`, `~~support platform`) resolved in `.mcp.json`, so you swap HubSpot→Salesforce without rewriting workflow logic ([CONNECTORS.md](https://raw.githubusercontent.com/anthropics/knowledge-work-plugins/main/customer-support/CONNECTORS.md), official).

| Plugin / skill | CSM repurpose | Source |
|---|---|---|
| `customer-support` → `customer-escalation` | Already triggers on *"customer threatening to churn"* / SLA breach; quantifies revenue risk, auto-routes to Leadership | [SKILL](https://raw.githubusercontent.com/anthropics/knowledge-work-plugins/main/customer-support/skills/customer-escalation/SKILL.md) (official) |
| `customer-support` → `customer-research` | Account-context engine: 5-tier source priority (internal KB → CRM → chat → web → inference) with confidence + gap flagging | [SKILL](https://raw.githubusercontent.com/anthropics/knowledge-work-plugins/main/customer-support/skills/customer-research/SKILL.md) (official) |
| `customer-support` → `draft-response` | Exec-comms: tone-by-situation matrix; quality gate blocks unauthorized commitments / roadmap leaks (critical for at-risk-account email) | [SKILL](https://raw.githubusercontent.com/anthropics/knowledge-work-plugins/main/customer-support/skills/draft-response/SKILL.md) (official) |
| `sales` → `pipeline-review` | Repurpose the 0–100, 4-dimension health score (activity recency flags 14-day silence, single-threading) as a **renewal-pipeline health scorer** | [SKILL](https://raw.githubusercontent.com/anthropics/knowledge-work-plugins/main/sales/skills/pipeline-review/SKILL.md) (official) |
| `sales` → `forecast` | Best/likely/worst + commit/upside with stage probabilities → **gross-retention / renewal forecast** | [SKILL](https://raw.githubusercontent.com/anthropics/knowledge-work-plugins/main/sales/skills/forecast/SKILL.md) (official) |

Fork these into a private `customer-success` plugin; **distribute via an admin-managed plugin marketplace, not per-user skill uploads** (governance — §3.4). Don't stack the productivity+sales+enterprise-search plugins "to be safe" — bloated tool sets are Anthropic's #1 documented agent failure mode ([effective-context-engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents), official).

### 1.2 Connector reality (the part the marketing skips)

The dominant CS platforms are **export-driven, not API-connected**, in Cowork today. Architect around CSV/export ingestion + human-gated write-back.

| CS tool | Cowork status (2026-05) | Source |
|---|---|---|
| Slack, Gmail, Google Drive/Calendar | Native, but **Google connectors are read-only** (Gmail = drafts only, no send; can't edit Sheets/Slides; no attachment access — container has no browser for OAuth write flow) | [medium/Kent Broadbent](https://kent-broadbent.medium.com/connecting-claudes-cowork-to-google-workspace-using-gws-f5057df35eb6) (community) |
| HubSpot, Zendesk/Intercom/Freshdesk | First-party / supported connector | [anthropic.com/news/claude-for-small-business](https://www.anthropic.com/news/claude-for-small-business) (official) |
| **Salesforce, Gainsight, Catalyst, Planhat** | **No turnkey native connector.** Salesforce only via Slack-MCP (chat context, not SF objects) or Einstein BYO-LLM; reachable via 3rd-party MCP (Composio) / aggregators (Common Room) | [eesel.ai](https://www.eesel.ai/blog/claude-cowork-salesforce-integration), [composio.dev](https://composio.dev/toolkits/salesforce/framework/claude-cowork) (community) |
| Gong / call transcripts | Via Composio MCP; Cowork **cannot join/record/transcribe calls** — needs an exported, speaker-attributed transcript | [tldv.io](https://tldv.io/blog/claude-cowork/) (community) |

Set destructive connector tools (`send_email`, `delete`) to **Ask**, read tools (`search_*`, `list_*`) to **Allow** ([productcompass.pm](https://www.productcompass.pm/p/claude-cowork-guide), community). Managed Google Workspace silently fails until an admin allowlists Claude under *Security → API controls → third-party app access* ([support.claude.com Google Workspace](https://support.claude.com/en/articles/10166901-use-google-workspace-connectors), official).

### 1.3 The four core CS workflows

All four exploit the same lever — **paste the whole corpus, not a sample, into one context** ([gtmaipodcast.com — Claude for CS](https://www.gtmaipodcast.com/p/claude-code-for-customer-success), community):

- **Churn / renewal risk:** build the model from *your* churned-account history, not a generic framework. Composite ≈ 60% internal (login-freq 30-day delta, feature depth, ticket sentiment, NPS, renewal proximity) + 40% external (headcount delta, **champion departure = strongest single predictor**, funding/runway). Score must be **deterministic and auditable** ([databar.ai](https://databar.ai/blog/article/claude-code-customer-success-churn-prevention), community).
- **QBR/EBR prep:** seed a project Memory with the account-roster CSV (account, tier, ARR, renewal date, owner); a QBR-prep skill turns a raw export into agenda + talking points + expansion hypotheses ([gtmaipodcast.com — Cowork for CS](https://www.gtmaipodcast.com/p/claude-cowork-for-customer-success), community).
- **Voice-of-customer:** batch over *all* NPS verbatims / tickets → top themes with representative quotes, churn-predictive language, top product gaps by frequency ([vitally.io](https://www.vitally.io/post/ai-prompts-for-cs), community).
- **Anti-hallucination gate (load-bearing):** never let Claude infer renewal dates or commercial terms from transcripts — add a global rule **"UNKNOWN, not inference"** plus a mandatory source-quote/citation column; cross-reference the CRM before any commitment ([tldv.io](https://tldv.io/blog/claude-cowork/), community).

### 1.4 Recurring rituals — and the reliability trap

Scheduled tasks: each run is a **fresh isolated session** with full skill/connector access, intervals hourly→weekly, **no memory persistence between runs** (write output to a file/Memory the next run reads) ([support.claude.com schedule](https://support.claude.com/en/articles/13854387-schedule-recurring-tasks-in-claude-cowork), official). But they **only run if the machine is awake and the Cowork view is focused** — a scheduled 3 PM task fired at 4:20 PM only on clicking into Cowork; missed runs get exactly **one** catch-up, the rest queue and fire simultaneously on launch ([github #36131](https://github.com/anthropics/claude-code/issues/36131), [#44128](https://github.com/anthropics/claude-code/issues/44128), community). **Route by failure cost:** low-stakes weekly health brief → Cowork scheduled task; anything SLA-/renewal-bound → **Cloud Routines** (runs on Anthropic infra, machine can be off) or external orchestration, not Cowork local tasks ([buildtolaunch.substack.com](https://buildtolaunch.substack.com/p/claude-cowork-scheduled-tasks-vs-routines-vs-loop), community).

---

## 2. Skills — the authoring delta

The basics (SKILL.md schema, <500 lines, progressive disclosure) are in `skills.md`. The non-obvious practitioner findings:

- **Triggering is closer to keyword/intent matching than semantic search.** Sandboxed evals: explicit-keyword prompts activate ~100%, conceptual phrasings ~0% — *"the problem is purely whether the model checks skills at all"* ([scottspence.com](https://scottspence.com/posts/measuring-claude-code-skill-activation-with-sandboxed-evals), community). So: pack the **user's literal vocabulary + near-synonyms** into the description; Anthropic's own skill-creator deliberately writes **"pushy"** descriptions because Claude *"undertriggers"* ([skill-creator SKILL.md](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md), official).
- **The description is a ~50-word routing trigger, not documentation.** Third person (it's injected into the system prompt), action verbs + concrete nouns/file types + literal trigger phrases + an explicit **NOT** boundary; frame as *"Load when [real user intent]"* ([research.perplexity.ai](https://research.perplexity.ai/articles/designing-refining-and-maintaining-agent-skills-at-perplexity), press/vendor-eng; [obra/superpowers](https://github.com/obra/superpowers/blob/main/skills/writing-skills/anthropic-best-practices.md), community). Kill abstractions ("helps with documents") — they fail routing every time ([mager.co](https://www.mager.co/blog/2026-03-08-claude-code-eval-loop/), community).
- **Two orthogonal failure axes, two gates.** A *quality* gate (output good when active?) and a *routing* gate (does it fire on the right intent?). Skills routinely pass one and fail the other ([mager.co](https://www.mager.co/blog/2026-03-08-claude-code-eval-loop/), community).
- **Eval-driven, not eyeballed.** ~20 trigger queries, **8–10 should-trigger + 8–10 should-not with deliberate near-miss negatives** (negatives matter more than positives); run each query **3×** (activation is stochastic — measure a rate); optimize the description on a **60/40 train/held-out split** to avoid overfitting; A/B compare blind (judge doesn't know which version) ([skill-creator SKILL.md](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md), official; [research.perplexity.ai](https://research.perplexity.ai/articles/designing-refining-and-maintaining-agent-skills-at-perplexity), press/vendor-eng).
- **Evals are a model-upgrade tripwire.** Re-run the suite before a new model reaches the team — catches quality regressions *and* skill obsolescence (base model absorbed the technique → delete it) ([claude.com/blog/improving-skill-creator](https://claude.com/blog/improving-skill-creator-test-measure-and-refine-agent-skills), official).
- **The "tax test" for every body line:** *would the agent get this wrong without it?* If not, delete. *"If your skill is easy to write, it is probably too long"* — explanatory prose belongs in `references/`, **one level deep** (Claude `head`s nested files) ([research.perplexity.ai](https://research.perplexity.ai/articles/designing-refining-and-maintaining-agent-skills-at-perplexity), press/vendor-eng; [obra/superpowers](https://github.com/obra/superpowers/blob/main/skills/writing-skills/anthropic-best-practices.md), community).
- **Adding a skill can silently break existing ones** (cross-skill contamination) — re-run the *whole* suite on any add ([research.perplexity.ai](https://research.perplexity.ai/articles/designing-refining-and-maintaining-agent-skills-at-perplexity), press/vendor-eng).
- **Composition decision rule:** reusable procedure → Skill; isolated context / different perms / cheaper model → sub-agent; live external system → connector/MCP; sub-skill **only when the parent is genuinely unwieldy** (premature decomposition is a named anti-pattern) ([skillsmith.app](https://www.skillsmith.app/blog/agent-skill-framework), community).
- **Security:** no signing/allowlisting/sandbox per skill — a skill is arbitrary instructions + scripts running with the agent's permissions. **Treat installing one like running an unaudited script** ([equipping-agents-with-agent-skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills), official).

---

## 3. Memory & Projects — the hygiene delta

Disambiguation of the four "memory" systems is in `memory.md`. Practitioner hygiene:

- **Write filter.** Memory holds only facts that *constrain future reasoning* — durable preferences, decisions (dated), failed approaches, stable account facts. **Do not** store anything re-fetchable from a connector (current ARR, open ticket count), tool output, relative dates, or secrets/PII — *"storing too much creates persistent context pollution"* ([effective-context-engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents), official).
- **Store / query / encode rule:** changes often & re-fetchable → connector query; stable & must survive sessions → Memory; a repeatable procedure → a **Skill**; a behavioral default for every task → **Instructions**. Memory stores *facts*, Skills store *how you work* ([cookbook: context engineering](https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools), official).
- **Project = the durable unit; `memory.md` is a hand-editable file in the project folder.** A **standalone folder session has no memory, instructions, or scheduled tasks** — file access only. That's the #1 "Claude forgot everything." Always use a Project for recurring account work ([support.claude.com projects](https://support.claude.com/en/articles/14116274-organize-your-tasks-with-projects-in-claude-cowork), official; [ryanandmattdatascience.com](https://ryanandmattdatascience.com/claude-cowork-projects/), community).
- **Structure:** one Project per high-touch strategic account (maximizes isolation, kills cross-account bleed); for a long tail of small accounts, one Project per CS *function* ("Renewals desk", "QBR factory") with per-account `account_<name>.md` context files attached ([nicholasrhodes.substack.com](https://nicholasrhodes.substack.com/p/claude-cowork-projects-business-os), community). Seed memory by re-running your top workflows once; force a fact with *"Save in memory that …"* ([ryanandmattdatascience.com](https://ryanandmattdatascience.com/claude-cowork-projects/), community).
- **Context-failure modes & countermeasures** ([dbreunig.com](https://www.dbreunig.com/2025/06/26/how-to-fix-your-context.html), community): poisoning (wrong fact saved, then steers everything → date-stamp facts, prune); distraction (hour-long session reuses history vs. reasons → fresh window on subject switch); clash (memory contradicts live data → delete contradicted facts, prefer connector for volatile data).
- **Memory rot:** absolute dates only (relative dates are the primary rot vector); cap size; schedule a recurring task that consolidates and de-conflicts `memory.md` per active account ([memory-tool docs](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool), official; [claudefa.st](https://claudefa.st/blog/guide/mechanics/auto-dream), community — auto-consolidation is documented in the Claude Code lineage; **Cowork parity unconfirmed**).
- **Connector-as-memory:** when durable knowledge must be team-shared / auditable / survive across projects, route it through a connector (e.g., Notion) instead of local `memory.md` ([karozieminski.substack.com](https://karozieminski.substack.com/p/claude-cowork-notion-connector-persistent-memory-tips), community, paywalled).
- **No session/result sharing** — treat the **Project folder as the bus**: inputs in, deliverables out, a `HANDOVER.md` referenced each session is the community substitute for cross-session/teammate handoff ([ruben.substack.com](https://ruben.substack.com/p/claude-cowork-project), community).

---

## 4. Workflow architecture & current-model prompting

Named patterns and blueprints are in `workflow-architecture-and-prompting.md`. The 2026 deltas:

### 4.1 The multi-agent debate is settled — and the answer is narrow

Cognition's 2026-04-22 reversal + Anthropic converge on one shape: **a single orchestrator owns the full context; sub-agents are ephemeral, isolated, read-only investigators that return a ≤1–2k-token distilled brief; all writes and consequential decisions stay single-threaded** ([cognition.ai/blog/multi-agents-working](https://cognition.ai/blog/multi-agents-working), community/primary-practitioner; [anthropic.com/engineering/multi-agent-research-system](https://www.anthropic.com/engineering/multi-agent-research-system), official). Encode that boundary in orchestrator skills: sub-agents *gather and analyze*; the main thread *decides and writes*. Run the QA/evaluator pass in a **clean-context sub-agent** (only the deliverable + rubric — *"shorter context for the reviewer improves intelligence via reduced context rot"*) ([cognition.ai](https://cognition.ai/blog/multi-agents-working), community).

**The number that governs every fan-out decision:** a single agent ≈ **4× chat tokens**, multi-agent ≈ **15×**, and **token spend explains ~80% of performance variance**. Fan-out scales with task class — 1 (lookup) / 2–4 (comparison) / 10+ (breadth-first research) — and every spawn must carry *objective + output format + tool guidance + explicit boundaries* or sub-agents duplicate each other ([anthropic.com/engineering/multi-agent-research-system](https://www.anthropic.com/engineering/multi-agent-research-system), official). **Fan out for breadth, never for depth** — a single deep risk score belongs on one thread.

### 4.2 Opus 4.7 / Sonnet 4.6 changed the defaults — rip out old scaffolding

Cowork runs the latest models; 2024–25-era scaffolding now actively hurts ([migration guide](https://platform.claude.com/docs/en/about-claude/models/migration-guide), [prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices), both official):

| New behavior | Remove | Do instead |
|---|---|---|
| More literal instruction-following | Assuming it generalizes a rule | State scope: *"apply to every section, not just the first"* |
| Fewer sub-agents/tools by default | Implicit "parallelize" | Say explicitly when to fan out and when not |
| Built-in progress updates | "summarize every 3 tool calls" | Delete it; describe desired updates with examples if wrong |
| Over-triggers on aggressive prompts | "CRITICAL: you MUST use this skill" | Plain "Use this when…"; if it under-fires, fix the *description*, don't shout |
| Strict effort calibration | Prompting around shallow reasoning | Raise effort to high/xhigh; don't prompt around it |
| New tokenizer (+up to 35% tokens) | Fixed token math / tight `max_tokens` | Re-baseline cost after any model upgrade |

The **reversibility clause** is the canonical safety primitive — lift verbatim into project instructions: *take local reversible actions freely; for hard-to-reverse / shared-system / externally-visible actions (send message, post to Slack, write the system of record, delete), ask first* ([prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices), official). It makes the model self-gate even when approval mode is permissive. Specify the whole task in **turn one** — drip-feeding context over turns reduces token efficiency and quality.

### 4.3 Observability

**OpenTelemetry is the only Cowork audit channel** (Team/Enterprise, opt-in, off by default). It emits per-prompt: full prompt text, every tool/MCP call + params, file paths, **which skills/plugins fired**, **approval decision per action**, tokens/cost; `prompt.id` correlates a whole trace ([support.claude.com OTel](https://support.claude.com/en/articles/14477985-monitor-claude-cowork-activity-with-opentelemetry), official). It ingests sensitive payloads by default — scope the OTLP endpoint and keep a complementary human-readable `outputs/what-changed.md` per run.

---

## 5. Security & data governance (the depth)

Headline facts are in `platform-deep-dive.md`. The non-obvious depth:

- **The sandbox is narrower than it sounds.** Only shell/code runs in the isolated VM. The **agent loop, file reads/writes in granted folders, web fetch, and local MCP servers run natively on the host** — and the VM is *deliberately opaque to your EDR* ([support.claude.com architecture](https://support.claude.com/en/articles/14479288-claude-cowork-desktop-architecture-overview), official). The VM protects the OS from Claude; it does **not** protect your files, and it is not the boundary for the lethal trifecta at all. Network-allowlist changes don't apply to in-flight sessions ([support.claude.com team/enterprise](https://support.claude.com/en/articles/13455879-use-claude-cowork-on-team-and-enterprise-plans), official).
- **Cowork is a textbook lethal-trifecta machine:** granted PII + untrusted content (a malicious ticket body / web page / Skill) + a connector that can send or a web-fetch that can encode-and-leak. Anthropic's stated post-mitigation prompt-injection rate on browser content is **~1% (non-zero)** ([support.claude.com safely](https://support.claude.com/en/articles/13364135-use-claude-cowork-safely), official). A **file-exfiltration via indirect prompt injection** (hidden text → `curl` to an attacker's Anthropic API key, which is whitelisted in the VM, **no approval requested**) was publicly disclosed **2026-05-18** and reported unpatched ([promptarmor.com](https://www.promptarmor.com/resources/claude-cowork-exfiltrates-files), community — re-check status). Guardrails are not a defense (*"99% is a failing grade"*) — **cut a leg**: connectors read-only (kills exfiltration, highest leverage), web search off unless required, dedicated empty workspace folder; never analyze externally-supplied docs/Skills in a session that also has sensitive folder access ([simonwillison.net lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/), [stop the lethal trifecta](https://simonwillison.net/2025/Sep/26/how-to-stop-ais-lethal-trifecta/), community). Meta's **Rule of Two**: never combine more than two of {untrusted input, sensitive data, state change/external comms} autonomously — if all three are required, force human approval ([simonwillison.net new papers](https://simonwillison.net/2025/Nov/2/new-prompt-injection-papers/), community).
- **OAuth over-grant is real and Anthropic-confirmed:** the Google Workspace consent screen requests **email-send** scope even though Claude only reads/drafts — a prompt-injected agent with that token *can send*. **The granted scope, not the advertised behavior, is the blast radius**; rotate/re-review tokens quarterly ([support.claude.com Google Workspace](https://support.claude.com/en/articles/10166901-use-google-workspace-connectors), official; [repello.ai](https://repello.ai/blog/claude-cowork-security), community).
- **The compliance gap, stated for a CS/legal org:** Cowork history is local-only, *"not subject to standard data retention policies and cannot be centrally managed or exported by admins,"* and **not captured in the Compliance API** ([support.claude.com team/enterprise](https://support.claude.com/en/articles/13455879-use-claude-cowork-on-team-and-enterprise-plans), official; corroborated [harmonic.security](https://www.harmonic.security/resources/securing-claude-cowork-a-security-practitioners-guide), community). Consequence: no central record of which customer files a session touched — legal hold / eDiscovery / GDPR erasure can't be served from Anthropic infra; per-machine FileVault/BitLocker is the *only* data-at-rest control. **Document that Cowork is not used for workflows requiring a regulatory audit trail (SOX/HIPAA/PCI/SOC 2) until audit coverage exists.** Commercial plans are not used for training and are deleted ~30 days post-deletion; **personal Pro/Max accounts bypass all governance — block them for work** ([privacy.claude.com retention](https://privacy.claude.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data), official).

### IT/InfoSec gate (hand to security before rollout)

- [ ] Enterprise tier (RBAC/spend require it; Team = org-wide on/off only; block personal accounts via SSO)
- [ ] SSO + SCIM; OTel → SIEM **wired and validated before rollout** (only Cowork visibility)
- [ ] Connectors default **read-only**; web search off unless justified; Chrome restricted to trusted sites
- [ ] FileVault/BitLocker enforced on every endpoint
- [ ] `Ask before acting` enforced for any session touching PII/financials
- [ ] Documented prohibition on regulated/eDiscovery workloads until audit coverage exists
- [ ] SIEM detections: MCP server outside allowlist · connector send/write/delete · credential-path reads (`~/.aws`, `~/.ssh`, `.env`) · any new Dispatch task · off-hours/anomalous session
(synthesized from official + [repello.ai](https://repello.ai/blog/claude-cowork-security), [harmonic.security](https://www.harmonic.security/resources/securing-claude-cowork-a-security-practitioners-guide), [elastic.co](https://www.elastic.co/security-labs/claude-code-cowork-monitoring-otel-elastic), community)

---

## 6. Cost & adoption

**Cost** (Pro/Team share one budget pool across chat+Code+Cowork; Cowork burns faster due to hidden screenshot/image tokens — [news.ycombinator.com](https://news.ycombinator.com/item?id=46612919), community):

- **Raw PDF ≈ 3,000 tokens/page; paste→Google Doc→`.md` ≈ <200/page** — the single biggest lever.
- **Plan/draft in chat, execute in Cowork** (avoid expensive iteration inside the agent loop); targeted edits ("only redo §3, no commentary"); upload recurring files to a Project once; keep search/connectors off by default ([ruben.substack.com 21 hacks](https://substack.com/@ruben/note/c-257696176), community).
- Decision heuristic non-coders get wrong ("Cowork = the easy one, use it for everything"): **claude.ai = think · Cowork = delegate artifact-heavy multi-step file work · Claude Code = anything repeatable/reliable/long** (Code is more reliable and cheaper in tokens for repeatable work) ([news.ycombinator.com](https://news.ycombinator.com/item?id=46612919), community).

**Adoption** (technical lead, non-technical team — [buildtolaunch.substack.com onboarding](https://buildtolaunch.substack.com/p/claude-onboarding-setup-guide), community):

- **Champion model, not big-bang:** one early adopter per role, 20-min walkthrough, real win, they teach peers. Track adoption by *whether champions modify plugins / create skills*, not seat count.
- **The "Day Two problem":** novelty dies day 2 — do a **Day 5–7 check-in, not Day 30**.
- **First-week mistakes:** skipping Project/folder setup; installing all plugins at once (context degradation, esp. Pro); top-down global instructions that don't fit real workflows (co-draft with champions); over-restricting permissions so it can't do anything.
- **Contrarian sequencing:** skip connectors weeks 1–2 (folder-only work, minimal troubleshooting), add them weeks 3–4 once usage is sustained.

---

## Sources

Trust order: Anthropic/Claude official > Agent Skills standard > Anthropic
press/vendor-eng > reputable press > independent community. Where official and
community disagree, the conflict is flagged inline.

**Official — Anthropic / Claude**

1. https://www.anthropic.com/product/claude-cowork — "consequential decisions remain with the user"
2. https://github.com/anthropics/knowledge-work-plugins (+ `/README.md`, `customer-support` & `sales` SKILL.md files, `CONNECTORS.md`) — fork-able CS/sales plugins, vendor-agnostic connector placeholders
3. https://www.anthropic.com/news/claude-for-small-business — HubSpot connector framing
4. https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents — context pollution, write filter, bloated-toolset failure
5. https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills — manual-audit security model for skills
6. https://www.anthropic.com/engineering/multi-agent-research-system — 4×/15× token economics, 80%-of-variance, fan-out rubric, delegation contract, eval discipline
7. https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md — "pushy" descriptions, eval loop, 60/40 split, blind A/B
8. https://claude.com/blog/improving-skill-creator-test-measure-and-refine-agent-skills — evals as model-upgrade tripwire
9. https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools — store/query/encode decision framework
10. https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool — memory hygiene, size/expiration, "usually refuses" sensitive data
11. https://platform.claude.com/docs/en/about-claude/models/migration-guide — Opus 4.7/Sonnet 4.6 behavioral deltas, new tokenizer
12. https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices — reversibility clause, don't-over-prompt-triggering
13. https://support.claude.com/en/articles/14116274-organize-your-tasks-with-projects-in-claude-cowork — project-scoped memory, instructions vs memory
14. https://support.claude.com/en/articles/13854387-schedule-recurring-tasks-in-claude-cowork — scheduled-task mechanics & limits
15. https://support.claude.com/en/articles/14479288-claude-cowork-desktop-architecture-overview — host-vs-VM split, VM opaque to EDR
16. https://support.claude.com/en/articles/13364135-use-claude-cowork-safely — ~1% prompt-injection rate, file-access discipline
17. https://support.claude.com/en/articles/13455879-use-claude-cowork-on-team-and-enterprise-plans — local history, Compliance-API exclusion, allowlist-on-new-session-only
18. https://support.claude.com/en/articles/14477985-monitor-claude-cowork-activity-with-opentelemetry — Cowork OTel events, `prompt.id`, opt-in
19. https://support.claude.com/en/articles/10166901-use-google-workspace-connectors — OAuth send-scope over-grant, org allowlist requirement
20. https://privacy.claude.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data — commercial ~30-day deletion, no default training

**Press / vendor-engineering**

21. https://research.perplexity.ai/articles/designing-refining-and-maintaining-agent-skills-at-perplexity — ≤50-word "Load when" descriptions, negatives-primacy, tax test, cross-skill contamination, Gotchas Flywheel

**Community**

22. https://www.gtmaipodcast.com/p/claude-code-for-customer-success & /claude-cowork-for-customer-success — whole-corpus churn model, roster-seeded QBR (paywalled past intro)
23. https://databar.ai/blog/article/claude-code-customer-success-churn-prevention — 60/40 composite score, champion-departure predictor
24. https://www.vitally.io/post/ai-prompts-for-cs — reusable QBR/VoC prompts (tool-agnostic)
25. https://tldv.io/blog/claude-cowork/ — least-privilege, no native CRM API, anti-hallucination gate, can't record calls
26. https://www.eesel.ai/blog/claude-cowork-salesforce-integration — Salesforce = no turnkey connector
27. https://composio.dev/toolkits/salesforce/framework/claude-cowork — 3rd-party MCP bridge
28. https://kent-broadbent.medium.com/connecting-claudes-cowork-to-google-workspace-using-gws-f5057df35eb6 — native Google connectors are read-only; container/OAuth root cause
29. https://www.productcompass.pm/p/claude-cowork-guide — Ask/Allow permission discipline
30. https://scottspence.com/posts/measuring-claude-code-skill-activation-with-sandboxed-evals — keyword-ish (not semantic) activation, measured rates
31. https://www.mager.co/blog/2026-03-08-claude-code-eval-loop/ — dual-gate harness, runs-per-query=3, before/after descriptions
32. https://github.com/obra/superpowers/blob/main/skills/writing-skills/anthropic-best-practices.md — third-person description rule, one-level-deep references
33. https://www.skillsmith.app/blog/agent-skill-framework — skill/sub-agent/sub-skill/MCP decision matrix, premature-decomposition anti-pattern
34. https://cognition.ai/blog/multi-agents-working — 2026-04-22 reversal: single-threaded writes, read-only sub-agents, clean-context reviewer
35. https://cognition.ai/blog/dont-build-multi-agents — share-context / actions-carry-decisions principles
36. https://ryanandmattdatascience.com/claude-cowork-projects/ — standalone-folder = no memory, editable `memory.md`, no scheduled-task fallback
37. https://nicholasrhodes.substack.com/p/claude-cowork-projects-business-os — per-account vs per-function project structure, seed-by-re-running
38. https://www.dbreunig.com/2025/06/26/how-to-fix-your-context.html — context poisoning/distraction/clash remediations
39. https://claudefa.st/blog/guide/mechanics/auto-dream — auto memory consolidation (Claude Code lineage; Cowork parity unconfirmed)
40. https://karozieminski.substack.com/p/claude-cowork-notion-connector-persistent-memory-tips — Notion-as-persistent-memory (paywalled)
41. https://ruben.substack.com/p/claude-cowork-project & https://substack.com/@ruben/note/c-257696176 — folder-as-bus handoff, 21 cost hacks (PDF→md, plan-in-chat)
42. https://www.promptarmor.com/resources/claude-cowork-exfiltrates-files — file-exfiltration-via-injection disclosure, dated 2026-05-18 (unpatched as written)
43. https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/ , /2025/Sep/26/how-to-stop-ais-lethal-trifecta/ , /2025/Nov/2/new-prompt-injection-papers/ — lethal trifecta, cut-a-leg, Rule of Two
44. https://repello.ai/blog/claude-cowork-security & https://www.harmonic.security/resources/securing-claude-cowork-a-security-practitioners-guide & https://www.elastic.co/security-labs/claude-code-cowork-monitoring-otel-elastic — connector blast radius, compliance-gap eDiscovery impact, OTel SIEM detections, IT checklist
45. https://github.com/anthropics/claude-code/issues/36131 & /44128 — scheduled-task focus bug, queue-and-fire, one catch-up
46. https://buildtolaunch.substack.com/p/claude-cowork-scheduled-tasks-vs-routines-vs-loop & /claude-onboarding-setup-guide — failure-cost task routing, champion model, Day-Two problem
47. https://findskill.ai/blog/claude-computer-use-cowork-setup/ — long-session/Dispatch failure-rate estimates (single-source), Windows disk trap
48. https://news.ycombinator.com/item?id=46612919 — shared budget pool, Cowork-vs-Code reliability/cost, exfil "click stop" critique

---

## Confidence & gaps

**Solid (Anthropic-official, verified this pass):** the host-vs-VM execution
split and EDR opacity; the Compliance-API / Audit-Log / Data-Export exclusion
and local-only history; OAuth send-scope over-grant on Google Workspace; the
~1% post-mitigation injection rate; scheduled-task machine-awake dependency;
the multi-agent token economics (4×/15×, ~80% variance) and fan-out rubric;
Opus 4.7/Sonnet 4.6 behavioral deltas and the reversibility clause; the
`knowledge-work-plugins` repo structure and that there is **no dedicated
customer-success plugin** (`customer-support`/`sales` are the fork bases);
the skill eval loop, "pushy" descriptions, and no-per-skill-signing security
model.

**Inference / community (directional — verify before stakeholder use):**

- **The whole CSM workflow layer** (churn-from-history, roster-seeded QBR,
  VoC batch synthesis, anti-hallucination gate) is community/practitioner
  craft, partly from paywalled posts and some written for "Claude Code for CS"
  — the mechanics transfer to Cowork but exact prompts/formats are
  reconstructed, not Anthropic-validated.
- **Connector landscape is the most volatile element.** "No turnkey
  Salesforce/Gainsight/Catalyst/Planhat connector" rests on community
  analysis + absence from official plugin lists; the directory grows weekly —
  re-verify at the live connectors directory before scoping a workflow on it.
  3rd-party MCP bridges (Composio, Common Room) were search-surfaced, not
  capability-validated.
- **Skill activation is "keyword-ish not semantic"** is an empirical
  black-box inference from one researcher against a specific model build;
  a working heuristic, not a documented guarantee, and shifts with model
  updates.
- **Memory auto-consolidation in Cowork is unconfirmed** — documented only in
  the Claude Code lineage ("Auto Dream"); treat Cowork `memory.md` hygiene as
  a manual burden until verified.
- **Single-source community estimates explicitly not measured benchmarks:**
  the long-session (~50%) / Dispatch (~70%) failure rates and the Windows
  `sessiondata.img` disk-fill string (findskill.ai only) — anecdotal.
- **The 2026-05-18 file-exfiltration disclosure** (promptarmor.com) is dated
  today and reported unpatched as written — re-check Anthropic's response
  status before acting on it.
- **Beta / "at this time" items** (re-verify before InfoSec sign-off):
  computer use (research preview, Pro/Max only), Dispatch/scheduled tasks
  (recent, carry the unresolved reliability bug), and the Compliance-API
  exclusion (worded "not captured **at this time**" — may change).

**Staleness:** researched **2026-05-18**; Cowork ships ~monthly. Connector
availability, RBAC/governance scoping, scheduled-task reliability, and any
beta-tagged item change fastest. Re-verify against the live Anthropic help
center and connectors directory before any stakeholder-facing commitment.

# Sources — consolidated index

*Every source cited across the `claude_rules` topic, tiered official vs community, with what it backs and which docs use it. Compiled 2026-06-04 from live web research; re-verify fast-moving items (path-scoped-rule bugs, auto-dream, version-gated keys) against the running Claude Code version before relying on them. 64 distinct sources: 32 official, 32 community.*

Tiering follows this repo's convention: **official** = Anthropic-controlled (`code.claude.com`, `platform.claude.com`, `claude.com/blog`, `anthropic.com`, the `anthropics/claude-code` issue tracker, MCP/Linux Foundation); **community** = practitioner blogs, academic preprints, and third-party tools. Official-issue-tracker entries are official *signals* (the report exists / its status), not documentation — their status churns.

---

## Official — Claude Code documentation (`code.claude.com/docs`)

| # | Source | Backs | Cited by |
|---|---|---|---|
| 1 | [How Claude remembers your project (memory)](https://code.claude.com/docs/en/memory) | The load-bearing page: CLAUDE.md locations/load-order, upward-walk + on-demand subdir loading, `@import` (inline, 4-hop), HTML-comment stripping, `.claude/rules/` mechanics + `paths:` globs + symlinks + user-vs-project priority, auto memory (v2.1.59+, 200-line/25KB), `claudeMdExcludes`, managed `claudeMd`, context-not-config, troubleshooting | **(all 8 docs)** |
| 2 | [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) | "Would removing this line cause a mistake?" test, include/exclude table, emphasis-as-tuning + over-specified failure pattern, hooks-are-deterministic | anatomy, adherence, audit |
| 3 | [Explore the .claude directory](https://code.claude.com/docs/en/claude-directory) | Rules-are-guidance-not-enforcement; 200-line migration trigger; `.claude/` layout | rules-directory |
| 4 | [Set up Claude Code in a monorepo or large codebase](https://code.claude.com/docs/en/large-codebases) | Two-level layering, starting-directory discipline, `claudeMdExcludes`, Read deny rules, sparse worktrees, per-directory skills, layering→plugins inflection, OTEL dead-skill detection | large-codebase, audit |
| 5 | [Explore the context window](https://code.claude.com/docs/en/context-window) | `/context`, illustrative startup figures, what-survives-`/compact` table, per-skill 5K/25K caps, subagent savings, 1M window | context, audit |
| 6 | [Manage costs effectively](https://code.claude.com/docs/en/costs) | Per-feature cost guidance, skills-on-demand remedy, prompt caching, enterprise cost baseline, `/compact` focus | context |
| 7 | [Understand context costs / features overview](https://code.claude.com/docs/en/features-overview) | The decision-system backbone: match-features-to-goal, compare-similar-features, per-feature cost table, reconciliation rules, "request vs guarantee", promotion ladder | choose, context, audit |
| 8 | [How Claude Code works](https://code.claude.com/docs/en/how-claude-code-works) | Auto-compaction order (tool outputs first, then summarize) | context |
| 9 | [Debug your configuration](https://code.claude.com/docs/en/debug-your-config) | `/memory`, `/context`, `/status`, `/doctor`, clean-config bisection | audit |
| 10 | [Automate actions with hooks](https://code.claude.com/docs/en/hooks-guide) | Hook blocking taxonomy, `PreToolUse` deny surviving bypass, `exit 2`/`permissionDecision`, prompt/agent hooks, settings-deny > hook-allow | choose, adherence, audit |
| 11 | [Hooks reference](https://code.claude.com/docs/en/hooks) | `InstructionsLoaded` payload (`memory_type`/`load_reason`), `ConfigChange` | anatomy, audit |
| 12 | [Extend Claude with skills](https://code.claude.com/docs/en/skills) | `disable-model-invocation`, `user-invocable`, `paths:`, description-budget truncation, `skillOverrides` | choose |
| 13 | [Create custom subagents](https://code.claude.com/docs/en/sub-agents) | Subagent startup load, scope precedence, inline `mcpServers`, `memory:` frontmatter (project = shareable) | choose, auto-memory, audit |
| 14 | [Connect Claude Code via MCP](https://code.claude.com/docs/en/mcp) | MCP scope precedence, Tool Search deferral | choose |
| 15 | [Create plugins](https://code.claude.com/docs/en/plugins) | Plugin packaging, `plugin:skill` namespacing, when-to-plugin | choose |
| 16 | [Settings reference](https://code.claude.com/docs/en/settings) | Settings precedence, `managed-settings.d/`, `claudeMd`, `claudeMdExcludes`, `strictPluginOnlyCustomization`, `outputStyle` | choose, large-codebase, audit |
| 17 | [Set up Claude Code for your organization](https://code.claude.com/docs/en/admin-setup) | Managed settings + lockdown switches, `/status` source tags, array-merge, hourly refresh | large-codebase, anatomy, audit |
| 18 | [Monitoring usage](https://code.claude.com/docs/en/monitoring-usage) | OTEL `skill.name`/`plugin_loaded` attribution for dead-config detection | audit |
| 19 | [Security](https://code.claude.com/docs/en/security) | Prompt-injection threat model, untrusted imports, `ConfigChange` audit hook | audit |
| 20 | [Claude Code changelog](https://code.claude.com/docs/en/changelog) | Used to confirm the `v2.0.64` rules attribution is *not* in the reachable changelog (only 2.1.121+) | rules-directory |

## Official — Anthropic platform, engineering & blog

| # | Source | Backs | Cited by |
|---|---|---|---|
| 21 | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) | Positive-over-negative framing, explain-the-why, positive examples, Opus 4.8 literalism / state-scope-explicitly | anatomy, adherence, audit |
| 22 | [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | Attention budget, smallest-high-signal-set, the context-rot recall mechanism | context |
| 23 | [How Claude Code works in large codebases (blog, 2026-05-14)](https://claude.com/blog/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start) | DRI/agent-manager roles, extension-priority order, LSP as highest-value, 3–6-month review cadence, agentic-search-not-RAG | large-codebase, audit |
| 24 | [Memory tool (API, `memory_20250818`)](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool) | The client-hosted `/memories` API tool — disambiguated from Claude Code auto memory | auto-memory |
| 25 | [Dreams (Managed Agents API, Research Preview)](https://platform.claude.com/docs/en/managed-agents/dreams) | Memory consolidation, non-destructive separate output store, beta headers | auto-memory |

## Official — issue tracker & ecosystem standards

| # | Source | Backs | Cited by |
|---|---|---|---|
| 26 | [Issue #5502 — CLAUDE.md ignored until asked](https://github.com/anthropics/claude-code/issues/5502) *(closed not-planned)* | Documents the user-message-not-system-prompt adherence gap | adherence, audit |
| 27 | [Issue #6973 — user-message vs `--append-system-prompt`](https://github.com/anthropics/claude-code/issues/6973) | The tier distinction and the system-prompt-priority lever | adherence |
| 28 | [Issue #16853 — path rules don't load in subdirs](https://github.com/anthropics/claude-code/issues/16853) *(open, v2.1.1)* | Path-scoped-rule reliability bug (never-loads) | rules-directory, audit |
| 29 | [Issue #16299 — path rules load globally](https://github.com/anthropics/claude-code/issues/16299) *(open, v2.0.76)* | The contradictory path-scoped-rule bug (loads-everywhere) | rules-directory, audit |
| 30 | [Issue #38487 — path rules blind to file creation](https://github.com/anthropics/claude-code/issues/38487) *(closed not-planned)* | Triggers on Read/Edit, not Write of a new file | rules-directory, audit |
| 31 | [Issue #37102 — auto memory pruning](https://github.com/anthropics/claude-code/issues/37102) *(closed not-planned)* | No shipped auto-prune; cleanup is manual | auto-memory, audit |
| 32 | [Issue #38461 — `/dream` returns Unknown skill](https://github.com/anthropics/claude-code/issues/38461) | Auto-dream not wired in Claude Code | auto-memory, audit |
| 33 | [Issue #38493 — dreamed memory unauditable](https://github.com/anthropics/claude-code/issues/38493) | Dreamed-memory accuracy/transparency gaps | auto-memory, audit |
| 34 | [Issue #21674 — `~/.claude/CLAUDE.md` injection vector](https://github.com/anthropics/claude-code/issues/21674) *(closed not-planned)* | Writable always-loaded user CLAUDE.md as a persistent injection vector | audit |
| 35 | [MCP knowledge-graph memory server](https://github.com/modelcontextprotocol/servers/tree/main/src/memory) | The third "memory" system — disambiguated | auto-memory |
| 36 | [Linux Foundation forms the Agentic AI Foundation (2025-12-09)](https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation) | AGENTS.md + MCP as AAIF project contributions | anatomy |

---

## Community — academic / empirical research

| # | Source | Backs | Cited by |
|---|---|---|---|
| 37 | [Evaluating AGENTS.md (ETH Zurich, arXiv 2602.11988)](https://arxiv.org/abs/2602.11988) | The experiment: repository-level context files can **reduce** success vs no context and add **>20%** inference cost; recommend minimal human-written files | adherence, context, audit |
| 38 | [InfoQ — coverage of the AGENTS.md value study](https://www.infoq.com/news/2026/03/agents-context-file-value-review/) | Secondary coverage of #37 (AGENTbench, step/token deltas) | adherence, context |
| 39 | [On the Use of Agentic Coding Manifests (arXiv 2509.14744)](https://arxiv.org/abs/2509.14744) | A **separate descriptive** study of 253 real CLAUDE.md files — explicitly *not* the source of the success/cost numbers | adherence |
| 40 | [IFScale — how many instructions can LLMs follow (arXiv 2507.11538)](https://arxiv.org/abs/2507.11538) | Instruction-budget decay: ~62–69% at 500 instructions, near-perfect at ~10, three decay families, primacy bias ~150–200 | adherence |
| 41 | [Context Rot (Chroma, 18-model study)](https://www.trychroma.com/research/context-rot) | Performance degrades as input length grows, well before overflow; distractors compound it | context |

## Community — tooling, linters & audit rubrics

| # | Source | Backs | Cited by |
|---|---|---|---|
| 42 | [agnix — linter/LSP/CI for CLAUDE.md/AGENTS.md/SKILL.md](https://github.com/agent-sh/agnix) | The closest thing to a deterministic CLAUDE.md grader; tiered autofix; rule taxonomy | audit |
| 43 | [cclint (felixgeelhaar)](https://github.com/felixgeelhaar/cclint) | Concrete thresholds + deprecated-model-ID check | audit |
| 44 | [cclint (carlrannaberg)](https://github.com/carlrannaberg/cclint) | CLAUDE.md + settings/hooks config linter | audit |
| 45 | [50-point Team Adoption Scorecard (digitalapplied)](https://www.digitalapplied.com/blog/claude-code-team-adoption-audit-50-point-scorecard-2026) | Maturity tiers, grades against repo evidence; the 1,100-line canonical-bad | audit |
| 46 | [100-point audit-prompt (FlorianBruniaux)](https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/tools/audit-prompt.md) | A runnable one-pass repo audit with point allocations + `paths:`-coverage credit + token-budget gate | audit |

## Community — practitioner write-ups

| # | Source | Backs | Cited by |
|---|---|---|---|
| 47 | [AGENTS.md open standard homepage](https://agents.md/) | 60k+ projects / 20+ tools, no required fields; the `@AGENTS.md`/symlink bridge | anatomy |
| 48 | [HumanLayer — Writing a good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md) | Tiny-root-file heuristic, `file:line` over snippets, ~150–200 instruction budget, "never send an LLM to do a linter's job" | anatomy, adherence, audit |
| 49 | [Thomas Wiegold — CLAUDE.md: helpful or expensive noise?](https://thomas-wiegold.com/blog/claude-md-helpful-or-expensive-noise/) | 95%→20–60% intra-conversation compliance decay; `uv` 160× tooling figure; secondary coverage of #37 | anatomy, adherence |
| 50 | [16x Eval — The Pink Elephant Problem](https://eval.16x.engineer/blog/the-pink-elephant-negative-instructions-llms-effectiveness-analysis) | The negative-instruction / ironic-process *anecdote* (no reliable percentage) | anatomy, adherence, audit |
| 51 | [Blink — CLAUDE.md best practices](https://blink.new/blog/claude-md-best-practices) | Specificity-as-safety (named prohibition beats vague category) | anatomy |
| 52 | [DEV — 5 patterns that make Claude follow rules](https://dev.to/docat0209/5-patterns-that-make-claude-code-actually-follow-your-rules-44dh) | Top/bottom primacy-recency placement of critical rules | anatomy, adherence |
| 53 | [Builder.io — Claude Code tips](https://www.builder.io/blog/claude-code) | In-session `#` add-to-memory shortcut auto-routing | anatomy |
| 54 | [Medium — /init brain transplant](https://medium.com/@AdithyaGiridharan/claude-codes-init-command-is-getting-a-brain-transplant-e0c389ec0c6b) | `CLAUDE_CODE_NEW_INIT=1` interview flow; `/init` ingesting AGENTS.md/.cursorrules | anatomy |
| 55 | [paddo.dev — Claude Code gets path-specific rules](https://paddo.dev/blog/claude-rules-path-specific-native/) | `v2.0.64` attribution; Cursor `.cursor/rules` comparison; frontmatter interop | rules-directory |
| 56 | [Modular Rules in Claude Code (setec.rs)](https://claude-blog.setec.rs/blog/claude-code-rules-directory) | Subdir organization; generic-filename anti-pattern | rules-directory, audit |
| 57 | [The Prompt Shelf — Claude Code in a monorepo](https://thepromptshelf.dev/blog/claude-code-monorepo-setup/) | Root sizing heuristics; per-package command scoping | large-codebase |
| 58 | [claudefa.st — large-codebase playbook](https://claudefa.st/blog/guide/development/large-codebase-playbook) | ~30k/~500k-line working thresholds (community estimates) | large-codebase |
| 59 | [claudefa.st — Claude Code Dreams](https://claudefa.st/blog/guide/mechanics/auto-dream) | CLAUDE.md-as-authoritative authority-split framing | auto-memory |
| 60 | [theaiarchitects — Subagents vs Skills](https://theaiarchitects.com/blog/claude-code-subagents-vs-skills) | The three subagent-vs-skill signals; "write a skill first" default | choose |
| 61 | [boringbot — Skills, Subagents, Hooks, Plugins](https://boringbot.substack.com/p/claude-code-skills-subagents-hooks) | Cost-vs-isolation spectrum; harness-vs-subagent orchestration separation | choose |
| 62 | [serendb — Claude Code's local memory is a security risk](https://serendb.com/blog/claude-code-local-memory-security-risk) | `~/.claude/projects` world-readable plaintext; transcripts capture secrets; grep recipe | auto-memory, audit |
| 63 | [wmedia.es — the /context command](https://wmedia.es/en/tips/claude-code-context-command-token-usage) | A real `/context` capture: system tools ~17.6K dwarf memory ~302 tokens | context, audit |
| 64 | [DAIR.AI — Does AGENTS.md actually help?](https://academy.dair.ai/blog/agents-md-evaluation) | Additional coverage of the AGENTS.md evaluation (available as alternate coverage of #37) | — (reference) |

---

## How this index was built

Two background research workflows ran on **2026-06-04**: (1) eight parallel agents each owning one slice (CLAUDE.md authoring, `.claude/rules/`, large-codebase governance, token economics, the mechanism-decision system, anti-patterns/adherence, auto memory, auditing), doing live `WebSearch`/`WebFetch` and returning tiered, source-cited findings; followed by (2) an adversarial verification pass that re-fetched the riskiest claims (version numbers, limits, precedence) against primary sources. That pass caught and corrected several errors now baked into the docs — most notably distinguishing the **experimental** AGENTS.md study (arXiv 2602.11988, #37) from the **descriptive** 253-file study (arXiv 2509.14744, #39), and stripping a fabricated "flipping negatives halved violations" figure from the pink-elephant source (#50). The seven content docs were then composed from the verified findings under a strict source-whitelist (no agent was permitted to cite a URL it had not fetched), and every doc was re-checked so that **100% of cited URLs trace to this index**.

## Confidence & gaps

- **Highest confidence:** the `code.claude.com/docs` and `platform.claude.com` pages (#1–25) were fetched directly on 2026-06-04 and are the spine of every mechanic stated as fact.
- **Status churns:** the nine `anthropics/claude-code` issues (#26–34) are real reports at fetch time, but open/closed status and the underlying behavior (especially the path-scoped-rule bugs #28–30 and auto-dream #32–33) can change release-to-release — re-verify against the running version.
- **Preprint / academic:** #37 and #40 are preprints/benchmarks, not peer-reviewed Anthropic results; treat their exact numbers as directional. #39 is a *descriptive* study and must not be conflated with #37's experiment.
- **Community estimates:** line-count heuristics (#48, #57), the `uv` 160× and 95%→20% decay figures (#49), the instruction-budget "~150–200 / ~50" framing (#48), and the `v2.0.64` rules version (#55) are practitioner/anecdotal — labeled as such wherever used, never presented as Anthropic facts.
- **Unfetched / lower confidence:** a few community pages (e.g. `claudelog.com`, the original announcement tweet) returned 403/402 during research and were reachable only via search snippets — they are **not** cited in any doc; nothing here depends on them.

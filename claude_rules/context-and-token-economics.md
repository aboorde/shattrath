# Context & Token Economics

*The token/context-budget lens on CLAUDE.md, rules, and memory: how to MEASURE what your config costs, what survives `/compact`, and why a fat-but-fitting CLAUDE.md still degrades behavior. Compiled 2026-06-04.*

## Mental model

Treat the context window as a **fixed budget** (200K default; 1M on Opus 4.6+/Sonnet 4.6 select-plan `[1m]` variants (source: https://code.claude.com/docs/en/context-window)). Every instruction token competes with the conversation, file reads, and tool output it is meant to help. Startup overhead — system prompt, built-in tools, CLAUDE.md, `MEMORY.md`, MCP tool names, skill descriptions — is paid on **every request** via prompt caching (source: https://code.claude.com/docs/en/costs), so a fat CLAUDE.md taxes the whole session. Two costs follow, not one: a **token cost** (every-request) and an **adherence cost** — "Longer files consume more context and reduce adherence" (source: https://code.claude.com/docs/en/memory). The discipline is **progressive disclosure**: keep CLAUDE.md the smallest set of always-true facts, push procedures to skills / path-scoped rules / subagents that load on demand, and **measure before trimming**.

> Auditor stance: do not trust intuition about where tokens go. Capture `/context`, find the dominant category, and cut *that*. The single largest fixed cost is almost never CLAUDE.md.

For the routing decision (which mechanism for which content) see [choosing-the-right-mechanism.md](choosing-the-right-mechanism.md); for the instruction-COUNT decay curve and adherence psychology see [adherence-and-anti-patterns.md](adherence-and-anti-patterns.md). This doc owns the budget.

---

## 1. MEASURE first — `/context` and `/memory`

The canonical way to audit what your config costs is to run it, not estimate it.

- **`/context`** — live per-category token breakdown with optimization suggestions (source: https://code.claude.com/docs/en/context-window).
- **`/memory`** — shows which CLAUDE.md and auto-memory files actually loaded at startup (source: https://code.claude.com/docs/en/context-window).
- The `InstructionsLoaded` hook logs which instruction files loaded, for scripted audits (see ground-truth load-order in [claude-md-anatomy.md](claude-md-anatomy.md)).

### Illustrative figures vs a real capture

The **official** interactive context-window visualization assigns *representative* startup costs — explicitly illustrative, not fixed (source: https://code.claude.com/docs/en/context-window):

| Startup item (official illustration) | Tokens |
|---|---|
| System prompt | 4,200 |
| Auto memory (`MEMORY.md`) | 680 |
| Environment info | 280 |
| MCP tool names (deferred) | 120 |
| Skill descriptions | 450 |
| User `~/.claude/CLAUDE.md` | 320 |
| Project `CLAUDE.md` | 1,800 |

The visualization's own note on Project CLAUDE.md: "Keep it under 200 lines. Move reference content to skills or path-scoped rules so it only loads when needed." (source: https://code.claude.com/docs/en/context-window)

Now a **real** `/context` capture (Opus 4.5, 200K window; community report, 2025-11) (source: https://wmedia.es/en/tips/claude-code-context-command-token-usage):

| Category (real capture) | Tokens | % of 200K |
|---|---|---|
| System prompt | 2.6K | 1.3% |
| **System tools (built-in)** | **17.6K** | **8.8%** |
| MCP tools | 907 | 0.5% |
| Custom agents | 935 | 0.5% |
| **Memory files (CLAUDE.md + auto memory)** | **302** | **0.2%** |
| Skills | 61 | 0.0% |
| Messages | 30.5K | 15.3% |
| Free space | 114K | 57% |
| Autocompact buffer | 33K | 16.5% |

**The lesson:** built-in **system tools (~17.6K) dwarf CLAUDE.md (~302 tokens)** — a ~58× gap in this capture. The official illustration's 1,800-token CLAUDE.md and 4,200-token system prompt do not match the real per-model numbers, which drift with version. The highest-leverage cuts are usually **disabling unused MCP servers and trimming the Messages / file-read column**, not shaving CLAUDE.md lines (source: https://wmedia.es/en/tips/claude-code-context-command-token-usage). Optimizing a 0.2% memory file while a 15% Messages column or 8.8% tool overhead goes untouched is wasted effort.

> Audit flag: a config review that recommends "shorten CLAUDE.md" without a `/context` capture showing CLAUDE.md is actually the dominant category is optimizing the wrong column.

---

## 2. Per-feature context-cost model (official)

The official features overview gives the per-mechanism cost model (source: https://code.claude.com/docs/en/features-overview); skills/MCP detail corroborated by the cost guide (source: https://code.claude.com/docs/en/costs):

| Mechanism | Loads when | What loads | Per-request cost |
|---|---|---|---|
| **CLAUDE.md** | Session start | **Full content** | **Every request** |
| **Skills** | Descriptions at start; body on use | Descriptions every request + full body when invoked | Low (descriptions) → body only when used |
| **MCP servers** | Names at start; schemas deferred | **Tool names only**, schemas deferred until used | Low until a tool is used |
| **Code intelligence (LSP)** | On lookup | Symbol results | Low, often **net-negative** (symbol lookup replaces file reads) |
| **Subagents** | On delegation | Isolated window | **Zero** main-context cost |
| **Hooks** | On trigger | Nothing (runs externally) | **Zero, unless** the hook returns additional context |

Key consequences:

- **Rules without `paths:` frontmatter** load at launch with the **same priority as `.claude/CLAUDE.md`** (source: https://code.claude.com/docs/en/memory) — so they carry the **same every-request cost as CLAUDE.md** and must be budgeted identically. **Path-scoped** rules (with a `paths:` glob) trigger only when Claude reads a matching file — not on every tool use. (Reliability gotchas for path-scoping live in [rules-directory.md](rules-directory.md).)
- **Subagents** are the primary lever to keep large reads off the main window: in the official example a research subagent read **6,100 tokens** of files but returned only a **420-token** summary to the parent — "That's the context savings." (source: https://code.claude.com/docs/en/context-window). The subagent loads its *own* CLAUDE.md copy (counts against ITS budget); the built-in Explore and Plan agents skip CLAUDE.md for a smaller context (source: https://code.claude.com/docs/en/context-window).
- **MCP is cheap idle**: schemas are deferred, so only names (~hundreds of tokens) load. The common "MCP is the big cost" assumption is wrong — system tools and accumulated file reads dominate (source: https://code.claude.com/docs/en/costs).

---

## 3. What survives `/compact` (official table)

Auto-compaction fires automatically near the limit: it clears **older tool outputs first**, then summarizes the conversation; requests and key code snippets are preserved, but **detailed early-conversation instructions may be lost** (source: https://code.claude.com/docs/en/how-claude-code-works). What gets re-injected from disk vs lost (official table) (source: https://code.claude.com/docs/en/context-window):

| Startup item | After `/compact` |
|---|---|
| System prompt + output style | Unchanged (not in message history) |
| **Project-root CLAUDE.md + unscoped rules** | **Re-injected from disk** |
| **Auto memory** (`MEMORY.md`) | **Re-injected from disk** |
| Rules **with** `paths:` frontmatter | **LOST** until a matching file is read again |
| **Nested** subdir CLAUDE.md | **LOST** until a file there is read again |
| **Skill DESCRIPTION index** | **LOST** — not re-injected; only invoked skills survive |
| Invoked **skill bodies** | Re-injected, **capped 5,000 tokens/skill, 25,000 total**, oldest dropped first |
| Hooks | N/A — run as code |

The skill description index is the **one startup item NOT re-injected** after `/compact`; only skills you actually invoked are preserved (visualization flag `noSurviveCompact: true` on skill descriptions) (source: https://code.claude.com/docs/en/context-window).

Practical diagnostic and remedies:

- An instruction that **disappears after `/compact`** was either conversation-only or lives in a nested CLAUDE.md that has not reloaded (source: https://code.claude.com/docs/en/memory).
- Anything that **must persist** across compaction belongs in **project-root CLAUDE.md** or an **unscoped rule** (drop the `paths:` frontmatter) (source: https://code.claude.com/docs/en/memory). For *must-run-at-a-point* behavior use a hook (or `--append-system-prompt` for system-prompt-level instructions) rather than betting a fat CLAUDE.md survives summarization (source: https://code.claude.com/docs/en/context-window).
- **Front-load** critical steps in `SKILL.md`: re-injected bodies truncate to 5,000 tokens/skill from the **top**, so the start of the file is what survives (source: https://code.claude.com/docs/en/context-window).
- Use `/compact focus on <X>` proactively at task boundaries (not reactively after quality drifts), and `/clear` at hard task boundaries (source: https://code.claude.com/docs/en/costs).

---

## 4. The 200-line rationale — cost AND adherence

Official size guidance: **target under 200 lines per CLAUDE.md**; "Longer files consume more context **and reduce adherence**." (source: https://code.claude.com/docs/en/memory). This is the load-bearing rule and it is **two reasons**, not one — token cost *and* instruction-following quality. CLAUDE.md is **loaded in full at session start regardless of length** and "consuming tokens alongside your conversation" on every request (source: https://code.claude.com/docs/en/memory).

CLAUDE.md vs `MEMORY.md` — different load semantics:

| File | Load semantics | Cap |
|---|---|---|
| **CLAUDE.md** | Full content, every session, every request | **No hard cap** — loaded in full regardless of length; 200 lines is a *target*, not a ceiling |
| **`MEMORY.md`** (auto memory) | First N loaded each session; topic files on demand | **Hard cap: first 200 lines OR 25KB, whichever comes first** — content beyond is not loaded at session start |

(source: https://code.claude.com/docs/en/memory). The 200-line/25KB cap **applies only to `MEMORY.md`** — CLAUDE.md is uncapped and full-loaded. Auto-memory storage, the three-memory disambiguation, and pruning discipline live in [auto-memory-and-self-improvement.md](auto-memory-and-self-improvement.md).

> Audit flag: a >200-line CLAUDE.md is not auto-truncated — it silently consumes its full length on every request. Flag it on both axes (budget + adherence), not just budget.

---

## 5. The `@import` "saves context" myth + progressive disclosure

`@path` imports are **expanded inline at launch** (max **4 hops**) (ground truth; see [claude-md-anatomy.md](claude-md-anatomy.md)). They help **organization** but do **NOT reduce context** — "imported files load at launch" alongside the referencing CLAUDE.md (source: https://code.claude.com/docs/en/memory). Splitting a bloated CLAUDE.md into `@imports` to "save tokens" is a no-op: the content still enters context on every request.

The **real** deferral levers (progressive disclosure):

| Lever | Defers what | Cost when idle |
|---|---|---|
| **Path-scoped rules** (`paths:` glob) | Directory/extension-specific guidance — loads only when a matching file is read | **Zero** on unrelated work (source: https://code.claude.com/docs/en/memory) |
| **Skills** | Full body; description in startup index | Description-only until invoked (source: https://code.claude.com/docs/en/costs) |
| **Skills with `disable-model-invocation: true`** | Body **and** description | **Zero** — kept out of the startup skill index entirely; the example `/commit-push` skill "cost zero context until this moment" (source: https://code.claude.com/docs/en/context-window) |
| **Subagents** | Bulk reads in an isolated window | Zero main-context cost; summary-only returns (source: https://code.claude.com/docs/en/context-window) |
| **Hooks** | Deterministic enforcement / pre-filtering | Zero unless they emit `additionalContext` (source: https://code.claude.com/docs/en/features-overview) |

The official remedy for "My CLAUDE.md is too large" is to **move detailed/specialized instructions OUT** into skills (on-demand) or path-scoped rules — "Skills load on-demand only when invoked, so moving specialized instructions into skills keeps your base context smaller." (source: https://code.claude.com/docs/en/costs). Reserve `@imports` purely for organizing **always-needed** content. Other zero-cost tricks: **block-level HTML comments** (`<!-- ... -->`) are stripped before injection, so maintainer notes/changelogs cost zero tokens (comments inside code blocks are preserved) (ground truth; see [claude-md-anatomy.md](claude-md-anatomy.md)); pre-filter verbose tool output with a `PreToolUse`/`PostToolUse` hook (e.g. `grep ERROR`) so a 10,000-line log enters context as hundreds of tokens (source: https://code.claude.com/docs/en/costs).

---

## 6. Context rot — why a fat-but-fitting CLAUDE.md still degrades behavior

A CLAUDE.md that *fits* the window is not "free." **Context rot** is the mechanism by which extra always-on tokens degrade behavior before any overflow.

- **Attention budget (official):** "LLMs have an 'attention budget' that they draw on when parsing large volumes of context… find the smallest possible set of high-signal tokens that maximize the likelihood of some desired outcome." (source: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents). Same source, the mechanism: "as the number of tokens in the context window increases, the model's ability to accurately recall information from that context decreases" — a **performance gradient, not a hard cliff**.
- **Chroma "Context Rot" study (community, 18 frontier models** incl. Claude 4 Opus/Sonnet, GPT-4.1, Gemini 2.5, Qwen3): all degrade as input length grows, "often in surprising and non-uniform ways," and degradation appears **well before the window is full** — distractors compound it; rot is distinct from overflow (source: https://www.trychroma.com/research/context-rot).
- **ETH Zurich AGENTS.md experiment (preprint):** *Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?* — Gloaguen, Mündler, Müller, Raychev, Vechev — found repository-level context files can **reduce success** vs no context while adding **>20% inference cost**, recommending human-written files "describe only minimal requirements" (source: https://arxiv.org/abs/2602.11988; coverage: https://www.infoq.com/news/2026/03/agents-context-file-value-review/). Treat the detailed sub-numbers (138-task AGENTbench, −0.5% SWE-bench Lite, +4% human-written AGENTbench, +14–22% reasoning tokens) as preprint/secondary, not settled Anthropic fact.

These three converge on the same operating rule: **more context ≠ better behavior**; curate the smallest high-signal set and offload bulk to subagents. The instruction-COUNT decay curve (IFScale, arXiv 2507.11538) and adherence psychology are out of lane here — see [adherence-and-anti-patterns.md](adherence-and-anti-patterns.md).

---

## 7. The 1M-window caveat + cost baseline

A bigger window does **not** escape context rot. The 1M-token window (Opus 4.6+/Sonnet 4.6 select plans; compaction works the same at the larger limit) (source: https://code.claude.com/docs/en/context-window) shifts the gradient but does not remove it. Chroma's own study — whose inputs top out around **~113K tokens** — shows degradation appears **well before the window is full** and that rot is distinct from window overflow (source: https://www.trychroma.com/research/context-rot). Community write-ups extrapolate this into rough rules of thumb — a clear effect somewhere around **300K–400K** on 1M-token models, and noticeable degradation possible by **~50K** on a 200K-window model — but those specific thresholds are **community extrapolation, presented as the author's own illustration and NOT stated in the Chroma report** (community: https://www.morphllm.com/context-rot). Filling a 1M window with rules and reads "because it fits" trades budget for behavior.

**Enterprise cost baseline (official):** ~**$13/developer/active day**, **$150–250/developer/month**, under **$30/active day for 90% of users** (source: https://code.claude.com/docs/en/costs). Token cost scales with context size; **prompt caching** reduces cost for repeated content like the system-prompt and CLAUDE.md prefix (source: https://code.claude.com/docs/en/costs) — which is exactly why a fat always-on prefix is paid on every request, cached or not.

---

## Auditor checklist (this doc's dimensions)

- [ ] A `/context` capture exists and the **dominant category was identified** before any CLAUDE.md trimming.
- [ ] CLAUDE.md is **≤200 lines** (target); flag longer on both budget and adherence.
- [ ] **Unscoped rules** are budgeted as every-request cost (same priority as CLAUDE.md); reference content is in **path-scoped rules / skills**, not CLAUDE.md.
- [ ] No `@import`-to-"save-context" pattern; imports used only for always-needed content.
- [ ] Side-effecting workflows use **skills** (ideally `disable-model-invocation: true`); bulk reads use **subagents**; enforcement uses **hooks**.
- [ ] Anything that must survive `/compact` is in **project-root CLAUDE.md or an unscoped rule**, not nested CLAUDE.md / path-scoped rules / conversation.
- [ ] `MEMORY.md` respected as **200-line/25KB hard-capped**; CLAUDE.md treated as uncapped/full-loaded.

Grading weights for these live in [audit-rubric.md](audit-rubric.md).

---

## Sources

1. https://code.claude.com/docs/en/context-window (official) — `/context`/`/memory` measurement, illustrative startup figures, what-survives-compaction table, skill body caps, 1M window, subagent savings.
2. https://code.claude.com/docs/en/memory (official) — CLAUDE.md full-load + every-request cost, 200-line/adherence rule, `@import` no-context-savings, `MEMORY.md` 200-line/25KB cap, unscoped-rule priority, compaction diagnostic.
3. https://code.claude.com/docs/en/costs (official) — per-feature cost guidance, skills-on-demand remedy, subagent/hook offload, prompt caching, enterprise cost baseline, `/compact` focus.
4. https://code.claude.com/docs/en/features-overview (official) — per-feature context-cost table (CLAUDE.md/skills/MCP/LSP/subagents/hooks).
5. https://code.claude.com/docs/en/how-claude-code-works (official) — auto-compaction order (tool outputs first, then summarize).
6. https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents (official) — attention budget, smallest-high-signal-set, context-rot recall mechanism.
7. https://www.trychroma.com/research/context-rot (community) — 18-model degradation study; degradation appears well before the window fills; rot distinct from overflow; tested inputs only up to ~113K tokens. (The 300K–400K / ~50K threshold figures are NOT in this report — see #11.)
8. https://arxiv.org/abs/2602.11988 (community/preprint) — ETH Zurich AGENTS.md evaluation: context files can reduce success, >20% cost.
9. https://www.infoq.com/news/2026/03/agents-context-file-value-review/ (community) — secondary coverage of the ETH AGENTS.md study.
10. https://wmedia.es/en/tips/claude-code-context-command-token-usage (community) — real `/context` capture (Opus 4.5, 200K): system tools ~17.6K vs memory ~302 tokens.
11. https://www.morphllm.com/context-rot (community) — origin of the 300K–400K (1M-window) and ~50K (200K-window) rule-of-thumb thresholds, presented there as the author's own illustration, not attributed to Chroma.

## Confidence & gaps

- **Verified official (safe as fact):** CLAUDE.md full-load + every-request cost; the 200-line target tied to *both* token cost and adherence; `@imports` do not reduce context; `MEMORY.md` 200-line/25KB cap (CLAUDE.md uncapped); the what-survives-`/compact` table and the skill-description index being the one item not re-injected; per-skill 5,000 / 25,000-token compaction caps; per-feature cost model; unscoped-rule = CLAUDE.md priority/cost; subagent 6,100→420 example; 1M window on Opus 4.6+/Sonnet 4.6; enterprise cost baseline; the attention-budget/context-rot framing from Anthropic engineering.
- **Illustrative, not fixed:** the official context-window visualization's per-category figures (Project CLAUDE.md 1,800; system prompt 4,200) are explicitly representative. The **real** numbers (system prompt 2.6K, system tools 17.6K, memory 302) are a **community** capture on **Opus 4.5 (2025-11)** and will drift with model/version — re-verify against a live `/context` run.
- **Community / preprint, labeled in-line:** Chroma's 18-model study (2025-07, Claude 4 era) is verified only for "all models degrade well before the window fills" and "rot ≠ overflow" — the specific **300K–400K / ~50K thresholds are NOT Chroma's** (its inputs topped out ~113K tokens); they are a secondary-blog (morphllm.com) extrapolation, and thresholds for Opus 4.7/4.8 are unverified and likely shifted. The ETH Zurich AGENTS.md >20%-cost / reduced-success result (preprint arXiv 2602.11988 — detailed sub-numbers are preprint/secondary). Do not attribute these to the separate descriptive study arXiv 2509.14744.
- **Version-sensitive / re-verify after major releases:** the per-skill 5,000 / 25,000-token caps, the 4-hop import depth, and the ~33K "autocompact buffer" (observed in one community capture; the exact auto-compact trigger % is not stated in official docs and may vary by model/window). The adherence-vs-length curve is asserted by official docs but **not quantified**.
- **Out of scope (cross-referenced, not duplicated):** instruction-COUNT decay (IFScale) and adherence psychology → [adherence-and-anti-patterns.md](adherence-and-anti-patterns.md); mechanism routing → [choosing-the-right-mechanism.md](choosing-the-right-mechanism.md); path-scoping reliability gotchas → [rules-directory.md](rules-directory.md); auto-memory storage/pruning → [auto-memory-and-self-improvement.md](auto-memory-and-self-improvement.md).

# Adherence & Anti-Patterns

_Why CLAUDE.md rules get ignored, the instruction-budget mechanics behind it, and a fix-tagged failure catalog for auditing a repo's Claude Code setup. Compiled 2026-06-04._

## Mental model

CLAUDE.md content is **delivered as a user message after the system prompt, treated as context, not enforced configuration** — Claude reads it and tries to follow it, but there is *no guarantee of strict compliance, especially for vague or conflicting instructions* (source: https://code.claude.com/docs/en/memory). Everything in this doc follows from that one fact: adherence is **probabilistic** and it is a **budget problem**. Instruction-following degrades roughly monotonically as the number of simultaneous instructions grows, and the longer a conversation runs the more the early instructions fade. So the audit posture is: count and prune instructions ruthlessly, reserve emphasis, phrase positively, resolve contradictions, reset the conversation at task boundaries — and migrate anything that *must* hold out of prose entirely. **Rules in prompts are requests; hooks in code are laws.** To block an action regardless of what Claude decides, use a `PreToolUse` hook (source: https://code.claude.com/docs/en/memory).

The auditor's one-line heuristic for every line of CLAUDE.md: *"Would removing this cause Claude to make mistakes? If not, cut it."* — and *bloated CLAUDE.md files cause Claude to ignore your actual instructions* (source: https://code.claude.com/docs/en/best-practices). For the routing decision (which mechanism a given rule belongs in) see [choosing-the-right-mechanism.md](choosing-the-right-mechanism.md); for scoring, [audit-rubric.md](audit-rubric.md).

## The instruction budget (why "more rules" backfires)

The strongest public proxy for CLAUDE.md adherence-vs-count is **IFScale** (arXiv 2507.11538, academic benchmark, community for our purposes), which measures instruction-following as a function of how many simultaneous instructions a model is given. The shape of that curve is the load-bearing finding; treat the exact percentages as model-version-specific and illustrative (the benchmark predates current Claude Code models).

| Instruction density | Behavior (IFScale) | Source |
|---|---|---|
| ~10 instructions | Near-perfect (98–100%) | arXiv 2507.11538 |
| ~100–150 | Top models begin dropping below 100% | arXiv 2507.11538 |
| 500 (max density) | Even best frontier models only ~62–69% (gemini-2.5-pro 68.9%, o3 62.8%, claude-3.7-sonnet 52.7%, gpt-4.1 48.9%) | arXiv 2507.11538 |

(source: https://arxiv.org/abs/2507.11538)

Three decay **families** — an audit can roughly map a model to one and predict where its CLAUDE.md collapses (source: https://arxiv.org/abs/2507.11538):

| Family | Models (illustrative) | Shape |
|---|---|---|
| Threshold decay | o3, gemini-2.5-pro | Hold near-perfect through ~150+, then decline |
| Linear decay | gpt-4.1, claude-3.7-sonnet | Steady, predictable decline from early on |
| Exponential decay | haiku, llama-4-scout | Rapid early collapse to a 7–15% floor |

**Primacy bias.** IFScale finds a measurable bias toward *earlier* instructions that peaks around **150–200 instructions**, then converges toward uniform failure (ratios ~1.0–1.5) at 300+ (source: https://arxiv.org/abs/2507.11538). The operational consequence is the most important one to internalize: at extreme density Claude does not merely drop the *last* rules — **it degrades across all of them uniformly**. This is the mechanism behind "I added a rule to fix behavior X and unrelated behavior Y silently regressed."

**Budget framing (community).** HumanLayer's practitioner estimate: frontier models follow **~150–200 discrete instructions** with reasonable consistency, and Claude Code's own system prompt already consumes **~50**, leaving ~100–150 slots for your rules — which argues for a ~30–80 line CLAUDE.md, not 200+ (source: https://www.humanlayer.dev/blog/writing-a-good-claude-md). Both numbers are community estimates, not an Anthropic benchmark; the 150–200 band is broadly consistent with IFScale's primacy-peak.

> **Audit move:** count *effective* instructions — every directive in CLAUDE.md **plus** `.claude/rules/` files without `paths:` **plus** every `@import` (all of which load at launch) — and treat the total as a hard ceiling. `@imports` do **not** lazy-load; they expand inline at launch (max 4 hops) and do not reduce the budget (source: https://code.claude.com/docs/en/memory). A common audit miss is counting imports as "free." See [context-and-token-economics.md](context-and-token-economics.md) for the full cost accounting.

Official corroboration without a curve: *target under 200 lines per CLAUDE.md file; longer files consume more context and reduce adherence* (source: https://code.claude.com/docs/en/memory). CLAUDE.md loads in **full** regardless of length, so size is a hard adherence cost, not a soft one.

## Compliance decay over a conversation (the freshness lever)

Adherence also decays *within a session* as context fills. An independent practitioner measurement (community, Thomas Wiegold) reports:

| Conversation position | Measured adherence |
|---|---|
| Messages 1–2 | ~95%+ |
| Messages 3–5 | 60–80% |
| Messages 6–10 | 20–60% (mostly absent past ten messages) |

(source: https://thomas-wiegold.com/blog/claude-md-helpful-or-expensive-noise/ — community/anecdotal)

The practical lever this implies: **`/clear` or a focused `/compact` at a task boundary restores CLAUDE.md authority more than any amount of emphasis does.** No quantity of uppercase recovers a context-saturated thread the way a reset does (source: https://thomas-wiegold.com/blog/claude-md-helpful-or-expensive-noise/).

**Official compaction fact (load-bearing for area rules):** after `/compact`, **project-root CLAUDE.md is re-read from disk and re-injected**, but **nested subdirectory CLAUDE.md is NOT re-injected automatically** — it reloads only the next time Claude reads a file in that subdir (source: https://code.claude.com/docs/en/memory). So an area-specific rule that was active early can silently vanish mid-session after a compaction. Truly always-on rules belong at project root or in always-loaded `.claude/rules/`; must-hold behaviors belong in a hook (compaction-independent).

## Emphasis is a scarce signal

Anthropic officially endorses emphasis as a tuning lever: *you can tune instructions by adding emphasis (e.g., "IMPORTANT" or "YOU MUST") to improve adherence* (source: https://code.claude.com/docs/en/best-practices). But it is framed as *tuning*, and the instruction-budget research explains why it scales poorly: emphasis works **only as a contrast against un-emphasized text**. If every rule is `IMPORTANT`/`YOU MUST`/caps, the model has no signal to prioritize and the emphasis becomes invisible — and piling on rules makes the model follow *all* of them worse (source: https://arxiv.org/abs/2507.11538).

- Reserve emphasis for the **1–2 genuinely load-bearing rules** (community consensus/inference — Anthropic gives no measured threshold).
- When a rule keeps getting ignored, the official diagnosis is *the file is probably too long and the rule is getting lost*, not insufficient emphasis (source: https://code.claude.com/docs/en/best-practices). Shorten or disambiguate first; add emphasis second; move to a hook if it must hold.
- Community placement pattern: duplicate the 2–3 most critical rules at the **top and bottom** of the file to exploit primacy/recency, rather than peppering `IMPORTANT` throughout (source: https://dev.to/docat0209/5-patterns-that-make-claude-code-actually-follow-your-rules-44dh).

## Positive vs. negative framing

Official prompt-engineering guidance, listed as the #1 success-criteria tip: *tell Claude what to do instead of what not to do* — and *positive examples … tend to be more effective than negative examples or instructions that tell the model what not to do* (source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices). Token generation is selection-positive: a prohibition only weakly suppresses.

The qualitative community anecdote ("pink elephant"): a CLAUDE.md saying *NEVER create duplicate files* still produced `file-fixed.py` / `file-correct.py`, attributed to an ironic-process / negation-resolution failure (source: https://eval.16x.engineer/blog/the-pink-elephant-negative-instructions-llms-effectiveness-analysis). _No reliable percentage exists for how much positive reframing helps; do not cite one._

Two reinforcing official points for current models: attach the **why** to every rule (*providing context or motivation … can help Claude better understand your goals*) and **state scope explicitly**, because Opus 4.8 *interprets prompts literally … does not silently generalize an instruction from one item to another* (source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices). Rewrite `Do NOT use default exports` → `Use named exports exclusively, in every module — tree-shaking depends on it.`

## The empirical case against bloat and auto-generation

The strongest evidence that *a bad context file is net-negative* is the ETH Zurich study **_Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?_** (Gloaguen, Mündler, Müller, Raychev, Vechev — arXiv 2602.11988, preprint). Its headline findings: context files can **reduce** task success versus no context and add **>20% inference cost**, and the recommendation is that human-written files *describe only minimal requirements* (source: https://arxiv.org/abs/2602.11988; coverage: https://www.infoq.com/news/2026/03/agents-context-file-value-review/; community secondary coverage: https://thomas-wiegold.com/blog/claude-md-helpful-or-expensive-noise/). In the community write-up, **Claude Code was the only agent where even developer-written files failed to improve performance** until pre-existing generic docs were removed (source: https://thomas-wiegold.com/blog/claude-md-helpful-or-expensive-noise/).

Implication for an auditor: **`/init` output committed verbatim is an anti-pattern.** Raw `/init` trends toward file-tree description and self-evident advice — exactly the categories the official include/exclude table says to cut. Treat `/init` as a draft, prune against that table, then commit. (A separate descriptive study, arXiv 2509.14744, catalogs the *structure* of 253 real CLAUDE.md files — do **not** attribute the success/cost numbers to it. See [claude-md-anatomy.md](claude-md-anatomy.md) for the include/exclude test in full.)

## Contradictions resolved arbitrarily

*If two rules contradict each other, Claude may pick one arbitrarily.* The official fix is to *review your CLAUDE.md files, nested CLAUDE.md files in subdirectories, and `.claude/rules/` periodically to remove outdated or conflicting instructions* (source: https://code.claude.com/docs/en/memory). Because discovery **concatenates** all found files (root→cwd, not override), a stale ancestor rule silently competes with a current one and behavior becomes nondeterministic and unauditable. Use `/memory` to enumerate everything loaded; use `claudeMdExcludes` to drop irrelevant ancestor/other-team files in monorepos.

## Stale model-workaround rules

A distinct, easily-missed bloat source: rules written to paper over a *past model's* quirk that the current model no longer has (e.g., "always re-read the file before editing," "don't write more than 50 lines at a time"). These fail the official inclusion test — *if Claude already does something correctly without the instruction, delete it or convert it to a hook* (source: https://code.claude.com/docs/en/best-practices) — yet they persist because nothing forces a re-test. **Audit move:** for each rule, ask whether removal causes a *currently reproducible* mistake on the model in use; delete the ones that only guarded against a retired model's behavior.

## "Never send an LLM to do a linter's job"

Rules like *always 2-space indent* or *no trailing whitespace* spend scarce instruction budget and per-session context on something a deterministic tool does for free, faster, and reliably (community: *never send an LLM to do a linter's job* — source: https://www.humanlayer.dev/blog/writing-a-good-claude-md). **Fix:** enforce formatting/lint via a `PostToolUse` hook that runs prettier/eslint after edits (or a pre-commit hook) and remove the rule from CLAUDE.md. Hooks are *deterministic and guarantee the action happens*, unlike advisory CLAUDE.md instructions (source: https://code.claude.com/docs/en/best-practices).

## The enforcement migration (the escape hatch)

Anything that **must** hold does not belong in prose. The official enforcement hierarchy, hardest to softest:

| Layer | Mechanism | Guarantee | Source |
|---|---|---|---|
| Managed deny | `permissions.deny` in **managed** settings; `claudeMd` in managed-settings.json | *deny rules from any settings scope, including managed settings, always take precedence over hook approvals*; managed CLAUDE.md/deny **cannot** be excluded by `claudeMdExcludes` | https://code.claude.com/docs/en/hooks-guide; https://code.claude.com/docs/en/memory |
| Hook deny | `PreToolUse` hook returning `permissionDecision: "deny"` (or exit code 2) | *blocks the tool even in bypassPermissions mode or with `--dangerously-skip-permissions`* — users cannot bypass via permission mode; hooks can tighten but not loosen | https://code.claude.com/docs/en/hooks-guide |
| Hook allow | `PreToolUse` allow | Auto-approves, but outranked by any settings `deny` | https://code.claude.com/docs/en/hooks-guide |
| Guidance | CLAUDE.md / rules | Advisory only — *shapes behavior but is not a hard enforcement layer* | https://code.claude.com/docs/en/memory |

Open issues confirm the failure this prevents: **#5502** documents Claude completing the work correctly but ignoring the CLAUDE.md workflow until explicitly asked "did you follow the CLAUDE.md instructions?" (closed not-planned — the fix is architectural, not a promised behavior change) (source: https://github.com/anthropics/claude-code/issues/5502). **#6973** is the root-cause thread: CLAUDE.md sits at the *user-message* tier, so for the handful of rules that truly need system-prompt priority, use `--append-system-prompt` (or managed `claudeMd`) instead (source: https://github.com/anthropics/claude-code/issues/6973). Canonical migration targets: a `protect-files.sh` `PreToolUse` hook that exits 2 on writes to `.env` / `package-lock.json` / `.git/`, a `block-rm-rf.sh` that denies dangerous Bash, and a `Stop` hook that runs the test suite and blocks the turn from ending until it passes (source: https://code.claude.com/docs/en/hooks-guide). Note the `Stop`-hook caveats: it must check `stop_hook_active` to avoid infinite loops, and Claude Code overrides it after **8 consecutive blocks** (raise via `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP`) (source: https://code.claude.com/docs/en/hooks-guide).

> **Audit flag:** any security- or safety-critical "rule" (never push to main, never edit migrations, never touch `.env`) living **only** in CLAUDE.md prose is a finding. Diagnose with the `InstructionsLoaded` hook (logs which instruction files load, when, and why) and `/memory`.

## Self-improving loops — with the write-only-knowledge warning

The official self-improving mechanism is **auto memory** (Claude-written, v2.1.59+): Claude writes its own learnings to `MEMORY.md`, of which the first 200 lines / 25KB load each session; topic files load on demand; it is machine-local and **manual-prune only** (see [auto-memory-and-self-improvement.md](auto-memory-and-self-improvement.md)). The heavier community equivalent is a **`Stop` hook that reads the session transcript, calls a model to extract patterns/mistakes/decisions, and writes them back** so the next session inherits them — the programmatic form of "add this to CLAUDE.md so you don't repeat the mistake" (source: https://www.mindstudio.ai/blog/self-evolving-claude-code-memory-obsidian-hooks).

**The warning is the point:** any append-only loop produces **write-only knowledge** that bloats the store and degrades the very adherence it was meant to improve — one community post documents **258 knowledge files loaded but never retrieved** (source: https://www.mindstudio.ai/blog/self-evolving-claude-code-memory-obsidian-hooks). A self-improving loop is only safe paired with **scheduled pruning/archival** and a calibrated extraction prompt that excludes obvious/one-time items. Capture without pruning is not a feature; it is the bloat anti-pattern on a timer.

## Anti-pattern catalog (with fixes)

| # | Anti-pattern | Why it fails | Fix | Source |
|---|---|---|---|---|
| 1 | Safety-critical "NEVER" rules in CLAUDE.md prose, trusted to hold | Context, not enforced config; probabilistic and dilutes/compacts away (#5502) | Move to `PreToolUse` hook (exit 2 / deny) or managed `permissions.deny`; CLAUDE.md restates rationale only | https://github.com/anthropics/claude-code/issues/5502; https://code.claude.com/docs/en/memory |
| 2 | Emphasis inflation — everything `IMPORTANT`/`YOU MUST`/caps | Emphasis only works as a scarce contrast; more rules → all followed worse | Reserve emphasis for 1–2 load-bearing rules; delete/downgrade the rest | https://code.claude.com/docs/en/best-practices; https://arxiv.org/abs/2507.11538 |
| 3 | Unbounded `/init` dumps + accreted "add this to CLAUDE.md" lines (kitchen sink) | Bloat makes Claude ignore real instructions; count drives the decay curve | Apply *"would removing this cause a mistake?"* per line; prune well under 200 lines; push area content to path-scoped rules/skills | https://code.claude.com/docs/en/best-practices |
| 4 | Negative / vague-category phrasing (*don't use mocks*, *never use any*, *don't change auth*) | Negation resolves poorly; vague rules under-apply under a literal model | Positive, specific directive + concrete example + named path (*don't touch `middleware.ts` without asking* > *don't change auth*) | https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices; https://eval.16x.engineer/blog/the-pink-elephant-negative-instructions-llms-effectiveness-analysis |
| 5 | Stale / self-contradicting rules across root, nested, `.claude/rules/`, user-level, left unreviewed | Conflicts resolved arbitrarily → nondeterministic, unauditable | `/memory` to enumerate; review periodically; `claudeMdExcludes` for stray ancestor/team files | https://code.claude.com/docs/en/memory |
| 6 | Treating `@path` imports as a way to keep CLAUDE.md "short" | Imports expand at launch (max 4 hops); zero budget reduction | Use path-scoped `.claude/rules/` (`paths:` frontmatter) or skills for sometimes-relevant content | https://code.claude.com/docs/en/memory |
| 7 | LLM rules doing a linter's job (indentation, whitespace) | Wastes budget/context on a deterministic, free tool | `PostToolUse` hook running prettier/eslint, or pre-commit hook; delete the rule | https://www.humanlayer.dev/blog/writing-a-good-claude-md |
| 8 | Append-only self-improving / `Stop`-hook memory loops | Write-only knowledge bloats and is never retrieved (258 files, 0 reads) | Pair with scheduled pruning + calibrated extraction prompt | https://www.mindstudio.ai/blog/self-evolving-claude-code-memory-obsidian-hooks |
| 9 | Assuming nested subdir CLAUDE.md rules persist after `/compact` | Only project-root is re-injected; nested reloads only on next read there | Always-on rules → project root or always-loaded rules; must-hold → hook | https://code.claude.com/docs/en/memory |
| 10 | Fixing a persistently-ignored rule by adding *more* emphasis | Usually signals the file is too long / phrasing ambiguous, not weak emphasis | Shorten file or disambiguate; if it must hold, move to a hook | https://code.claude.com/docs/en/best-practices |
| 11 | Embedded code snippets / pasted API docs / file-by-file descriptions | Go stale fast, are inferable, consume always-loaded budget | `file:line` references + links; progressive disclosure to `agent_docs/*.md` | https://www.humanlayer.dev/blog/writing-a-good-claude-md |
| 12 | Stale model-workaround rules guarding against a retired model's quirk | Fail the inclusion test; persist because nothing re-tests them | Delete any rule whose removal does not cause a *currently reproducible* mistake | https://code.claude.com/docs/en/best-practices |

## Sources

1. https://code.claude.com/docs/en/memory — (official) CLAUDE.md is a user message / context not enforced config; compaction re-injection; contradictions arbitrary; `@import` 4-hop expansion; enforcement-vs-guidance split.
2. https://code.claude.com/docs/en/best-practices — (official) inclusion test; bloat causes ignored instructions; over-specified-CLAUDE.md failure pattern; emphasis tuning; hooks deterministic; delete-if-already-correct.
3. https://code.claude.com/docs/en/hooks-guide — (official) `PreToolUse` deny survives bypass modes; settings-deny outranks hook approvals; `Stop`-hook gate, `stop_hook_active`, 8-block cap.
4. https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices — (official) positive over negative framing; positive examples beat negatives; attach the why; Opus 4.8 literalism / state scope.
5. https://arxiv.org/abs/2507.11538 — (community/academic) IFScale instruction-budget decay: ~62–69% at 500, near-perfect at ~10, three decay families, primacy bias ~150–200.
6. https://arxiv.org/abs/2602.11988 — (community/preprint) ETH Zurich *Evaluating AGENTS.md*: context files reduce success / >20% cost; recommend minimal human-written files.
7. https://arxiv.org/abs/2509.14744 — (community/academic) descriptive study of 253 CLAUDE.md files (referenced as *not* the source of the success/cost numbers).
8. https://www.infoq.com/news/2026/03/agents-context-file-value-review/ — (community) coverage of the ETH Zurich study.
9. https://thomas-wiegold.com/blog/claude-md-helpful-or-expensive-noise/ — (community) 95%→20–60% intra-conversation compliance decay; secondary coverage of ETH Zurich findings.
10. https://www.humanlayer.dev/blog/writing-a-good-claude-md — (community) instruction-budget framing (~150–200 / ~50 system); "never send an LLM to do a linter's job"; file:line over snippets.
11. https://eval.16x.engineer/blog/the-pink-elephant-negative-instructions-llms-effectiveness-analysis — (community) pink-elephant negative-instruction anecdote.
12. https://www.mindstudio.ai/blog/self-evolving-claude-code-memory-obsidian-hooks — (community) `Stop`-hook self-improving loop; 258-files-never-retrieved write-only-knowledge warning.
13. https://dev.to/docat0209/5-patterns-that-make-claude-code-actually-follow-your-rules-44dh — (community) top/bottom primacy-recency placement of critical rules.
14. https://github.com/anthropics/claude-code/issues/5502 — (official issue, closed not-planned) CLAUDE.md workflow ignored until explicitly prompted.
15. https://github.com/anthropics/claude-code/issues/6973 — (official issue) user-message vs system-prompt tier; `--append-system-prompt` as the lever.

## Confidence & gaps

**Verified official (safe to state as fact):** CLAUDE.md is a user message after the system prompt and is context, not enforced configuration, with no compliance guarantee; the inclusion test and "bloat causes ignored instructions"; emphasis (`IMPORTANT`/`YOU MUST`) as an endorsed *tuning* lever; contradictions resolved arbitrarily; positive-over-negative framing and "attach the why" and Opus-4.8 literalism; the compaction asymmetry (root re-injected, nested not); the enforcement hierarchy (managed deny > hook deny > hook allow > guidance) and that `PreToolUse` deny survives bypass modes; `Stop`-hook 8-block cap and `stop_hook_active`; `@import` 4-hop launch-time expansion; the <200-line target. (Sources 1–4, 14–15.)

**Community / inference — do not present as Anthropic facts:**
- **IFScale (2507.11538)** is a keyword-inclusion business-report benchmark across many providers (incl. claude-3.7-sonnet); it is the best public proxy for instruction-budget degradation but is **not** a Claude Code CLAUDE.md study and predates Opus 4.7/4.8. Treat the **shape** (monotonic decay, three families, ~150–200 primacy peak, uniform degradation at extreme density) as robust; treat the **exact percentages** as version-specific/illustrative and likely stale for current models.
- The **~150–200-follow / ~50-in-system-prompt** budget figures (HumanLayer) are estimates, not a published Anthropic benchmark — heuristics, not measured limits.
- The **95%→20–60% intra-conversation decay**, the **uv-named-160×** tooling figure, and the **ETH Zurich 20%/4%/2.7%** secondary numbers are community/anecdotal (Wiegold) restatements; the ETH paper (2602.11988) is a preprint and its detailed sub-numbers should be treated as preprint+secondary, not settled.
- **"Reserve emphasis for 1–2 rules"** is community consensus/inference — Anthropic endorses emphasis but publishes **no** measured efficacy data or count threshold.
- The **pink-elephant** anecdote is real and qualitative; **no** percentage for "positive reframing cut violations by X" is supported — none is cited here, by design.
- **Issue #5502** closed not-planned with no maintainer technical response; the remedy is architectural (hooks / `--append-system-prompt`), not a promised CLAUDE.md priority change. Re-verify whether newer Claude Code versions raised CLAUDE.md priority.

**Version-sensitive / re-verify (fast-moving, as of 2026-06-04):** IFScale percentages vs current Claude models; whether positive-framing gains hold specifically on Opus 4.8 (no A/B found); the `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` default (8) and auto-memory cap (200 lines / 25KB, v2.1.59+) are current-version facts that may shift across releases.

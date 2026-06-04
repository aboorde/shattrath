# Claude Code instruction architecture — best practices & audit kit

A technically-deep, **exhaustively source-linked** brief on getting the most out of **`CLAUDE.md` and `.claude/rules/`** in large codebases — plus a runnable **audit rubric** for grading an existing repo against best practice. Written for an **Opus coding agent** (and the power users who direct it): no "what is CLAUDE.md" 101, only advanced, measured, *checkable* techniques. The intended use is to hand this whole folder to an agent and say *"compare this repo's Claude Code setup to these best practices."*

Every non-obvious claim carries an inline source URL. Each doc ends with a numbered `## Sources` list and a `## Confidence & gaps` section separating Anthropic-official facts from community/inference and flagging staleness. **100% of cited URLs trace to [sources.md](sources.md)** — no agent was permitted to cite a page it had not fetched.

## How it was made (methodology)

Compiled **2026-06-04** by two background research workflows. Claude Code's memory system moves fast (the entire `.claude/rules/` feature, auto memory, and `managed-settings.d/` postdate the assistant's training cutoff), so everything was verified against **live web research**, prioritizing official Anthropic sources (`code.claude.com/docs`, `platform.claude.com`, `claude.com/blog`, the `anthropics/claude-code` issue tracker).

1. **Research fan-out (8 parallel agents)** — one per slice (CLAUDE.md authoring, `.claude/rules/`, large-codebase governance, token economics, the mechanism-decision system, anti-patterns/adherence, auto memory, auditing), each returning tiered, source-cited findings.
2. **Adversarial verification** — a second pass re-fetched the riskiest claims (version numbers, line/token limits, precedence, absolute words) against primary sources. It caught real errors now corrected here — e.g. distinguishing the **experimental** AGENTS.md study (arXiv 2602.11988) from a **separate descriptive** one (arXiv 2509.14744), and removing a fabricated statistic from a community source.

Because this is a fast-moving area, **re-verify anything tagged version-sensitive or community before relying on it** — especially the path-scoped-rule reliability bugs and the `Dreams`/auto-dream status, which churn release-to-release. Where official docs and community reports disagree, the docs say so rather than papering over it.

## The documents

| Doc | What it covers | Read it for |
|---|---|---|
| [cheatsheet.md](cheatsheet.md) | One-page printable distillation of the whole set | Print-and-go quick reference |
| [audit-rubric.md](audit-rubric.md) | **The deliverable** — a scored, ID'd checklist (severity, detection, remediation) an agent runs against a repo; static-vs-runtime split; an output format | Grading a repo's setup against best practice |
| [claude-md-anatomy.md](claude-md-anatomy.md) | The `CLAUDE.md` file: every load location & order, the discovery walk, `@imports` (inline/4-hop), HTML-comment stripping, AGENTS.md interop, `/init`, authoring-for-adherence | How CLAUDE.md actually loads & how to write a kept line |
| [rules-directory.md](rules-directory.md) | `.claude/rules/`: recursive discovery, `paths:` globs, priority, symlinks, naming taxonomy, the **reliability gotchas**, Cursor comparison, rule-vs-CLAUDE.md-vs-skill decision | What goes in `.claude/rules/` and when to trust `paths:` |
| [choosing-the-right-mechanism.md](choosing-the-right-mechanism.md) | The **decision system**: a master table routing any instruction by load-timing / context-cost / guarantee-vs-guidance across all ~16 mechanisms; the enforcement escape hatch; the promotion ladder | "Where does this instruction belong?" — the hub doc |
| [context-and-token-economics.md](context-and-token-economics.md) | Measuring config cost (`/context`), the per-feature cost table, what survives `/compact`, the 200-line rationale, the `@import` myth, context rot (IFScale/Chroma/ETH) | The token-budget lens; why a fat-but-fitting file still hurts |
| [adherence-and-anti-patterns.md](adherence-and-anti-patterns.md) | Why CLAUDE.md gets ignored: instruction-budget mechanics, compliance decay, emphasis discipline, positive framing, a fix-tagged anti-pattern catalog, the enforcement migration | "Why isn't my CLAUDE.md being followed?" |
| [large-codebase-patterns.md](large-codebase-patterns.md) | Monorepo/large-repo: two-level layering, starting-directory discipline, `claudeMdExcludes`, LSP, sparse worktrees, plugin centralization, org governance (managed policy, lockdown switches, DRI, OTEL dead-skill detection) | Scaling and governing instructions across a big tree |
| [auto-memory-and-self-improvement.md](auto-memory-and-self-improvement.md) | Auto memory (the Claude-written sibling): the three-memory disambiguation, index discipline, the manual-prune reality, promotion to CLAUDE.md, subagent memory, `Dreams` status, security | The other memory system, and self-improving loops done safely |
| [sources.md](sources.md) | The consolidated, tiered (official/community) source index with what each backs | Checking provenance |

## Two questions this answers up front

**"Is `.claude/rules/` real, and how does it differ from `CLAUDE.md`?"** → Yes — it is an official Claude Code feature. It is `CLAUDE.md` decomposed into per-topic `.md` files. A rule **without** `paths:` frontmatter loads at launch at the **same priority and the same every-request cost as `.claude/CLAUDE.md`**; a rule **with** a `paths:` glob is *meant* to load only when Claude reads a matching file (source: https://code.claude.com/docs/en/memory). Two cautions an auditor must know: (1) **`@imports` do NOT save context** — they expand inline at launch (max 4 hops), so splitting a file via `@import` is organization, not deferral; the real deferral levers are path-scoped rules, skills, and subagents; (2) **path-scoped triggering is version-fragile** — it has shipped broken both ways (never-loads #16853, loads-globally #16299) and is blind to file *creation* (#38487), so verify with `/memory` before trusting it. Full detail in [rules-directory.md](rules-directory.md).

**"Why isn't my `CLAUDE.md` being followed?"** → Because it is **not enforcement.** CLAUDE.md is delivered as a **user message after the system prompt** and is *"context, not enforced configuration… no guarantee of strict compliance"* (source: https://code.claude.com/docs/en/memory). Adherence is a budget that degrades with instruction count and conversation length (IFScale; ETH Zurich's finding that bloated/auto-generated context files can *reduce* success and add >20% cost). The fixes: ruthlessly prune (the "would removing this line cause a mistake?" test), phrase positively and specifically, reserve emphasis, and — the load-bearing move — **migrate anything that must hold into a `PreToolUse` hook or `permissions.deny`.** *Rules in prompts are requests; hooks in code are laws.* Full detail in [adherence-and-anti-patterns.md](adherence-and-anti-patterns.md) and [choosing-the-right-mechanism.md](choosing-the-right-mechanism.md).

## One-screen mental model

- **Three axes route everything.** Place any instruction by *when it loads* (always / path-matched / on-demand / event-driven / enforced-by-client), its *context cost* (every-request / cheap-until-used / isolated / zero), and whether you need a *guarantee or guidance*. Only **hooks and `settings.permissions`** are guarantees; CLAUDE.md, rules, imports, skills, and auto memory are all guidance.
- **CLAUDE.md is a finite, lossy instruction budget**, loaded in full every session (target **<200 lines**). Keep only non-inferable, behavior-shifting facts. Push file-type guidance to **path-scoped rules**, procedures/reference to **skills**, heavy reads to **subagents**, and must-hold rules to **hooks**.
- **Don't conflate the layers.** *CLAUDE.md/rules* = human-written, version-controlled, team-shared instructions · *auto memory* = Claude-written, machine-local, manually-pruned learnings (promote a stabilized fact up into CLAUDE.md to share it) · *hooks/permissions* = deterministic enforcement · *skills/MCP/subagents* = on-demand capability.
- **In a monorepo, one root CLAUDE.md is an anti-pattern.** Two-level layering (root = repo-wide <200 lines; per-package = local, owned by directory owners) + **starting-directory discipline** (launch inside the package a task touches) is the primary, zero-config scoping lever.
- **Measure before trimming.** Run `/context` — built-in system tools and file reads usually dwarf CLAUDE.md. Optimizing a 0.2% memory file while a 15% message column goes untouched is wasted effort.
- **To audit a repo, run [audit-rubric.md](audit-rubric.md).** Lead with Dimension A (enforcement written as prose = the top finding), separate static checks from runtime ones (`/memory`, `/context`, `/status`, OTEL), and rank fixes by severity × blast radius.

## Caveats

Researched **2026-06-04**. Claude Code ships on a rapid cadence; version-gated features (auto memory v2.1.59+, `managed-settings.d/` v2.1.83+, and others) and the path-scoped-rule bugs and `Dreams`/auto-dream status change fastest. Treat community figures (line-count heuristics, the `uv` 160× and compliance-decay numbers, the `v2.0.64` rules version) as directional only. Each doc's `## Confidence & gaps` is the authoritative statement of what is solid vs. inferred, and [sources.md](sources.md) tiers every source.

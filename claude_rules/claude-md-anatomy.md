# CLAUDE.md Anatomy & Mechanics

*The CLAUDE.md file itself — every load location, the discovery walk, `@path` imports, AGENTS.md interop, and authoring for adherence. Compiled 2026-06-04; mechanics are version-sensitive (re-verify against the changelog).*

## Mental model

CLAUDE.md is delivered as a **user message after the system prompt** — it is *context, not enforced configuration*, with no compliance guarantee (source: https://code.claude.com/docs/en/memory). Treat it as a finite, lossy instruction budget that is loaded in **full** every session and whose adherence decays both as the file grows and as the conversation lengthens. Author it like code under one test: **"would removing this line cause a specific mistake?"** — if not, cut it, because bloat causes Claude to ignore the rules that matter (source: https://code.claude.com/docs/en/best-practices). When a rule must hold *every time*, it does not belong in CLAUDE.md at all — route it to a hook (deterministic) or a path-scoped rule/skill (loaded only when relevant). This doc owns the file's mechanics; the **cost** angle lives in [context-and-token-economics.md](context-and-token-economics.md), deep **failure-mode** analysis in [adherence-and-anti-patterns.md](adherence-and-anti-patterns.md), modular splitting in [rules-directory.md](rules-directory.md), and the routing decision in [choosing-the-right-mechanism.md](choosing-the-right-mechanism.md).

## 1. Every CLAUDE.md location, scope & load order

Files are discovered and **concatenated** (never overridden) into context, ordered **broadest scope → most specific**; the project file therefore appears *after* the user file, and the file closest to cwd appears last (source: https://code.claude.com/docs/en/memory). All are loaded in full regardless of length.

| Tier | Path(s) | Scope / who shares it | Load timing | Excludable? |
|---|---|---|---|---|
| **Managed policy** | macOS `/Library/Application Support/ClaudeCode/CLAUDE.md`; Linux/WSL `/etc/claude-code/CLAUDE.md`; Windows `C:\Program Files\ClaudeCode\CLAUDE.md`; or `claudeMd` key in `managed-settings.json` | Org-wide, pushed by admins; every user on the machine | At launch, first (broadest) | **No** — cannot be excluded (source: https://code.claude.com/docs/en/memory; source: https://code.claude.com/docs/en/admin-setup) |
| **User** | `~/.claude/CLAUDE.md` | The individual, across *all* their projects; machine-local, not committed | At launch, after managed | Yes |
| **Project** | `./CLAUDE.md` **or** `./.claude/CLAUDE.md` | The team; committed to the repo | At launch, after user | Yes (via `claudeMdExcludes`) |
| **Local** | `./CLAUDE.local.md` | One developer's per-repo overrides; gitignored | At launch, last (most specific) | Yes |
| **Subdirectory** | `<subdir>/CLAUDE.md`, `<subdir>/CLAUDE.local.md` | That subtree; team or local | **On demand** — only when Claude reads files in that subdir | n/a |

- **`CLAUDE.local.md`** loads alongside `CLAUDE.md` and is treated the same way; `/init`'s personal option creates it and gitignores it for you. One caveat: a gitignored `CLAUDE.local.md` only exists in the worktree where it was created, so to share personal prefs **across git worktrees** the official guidance is to `@`-import a file from your home directory instead (e.g. `@~/.claude/my-project-instructions.md`) (source: https://code.claude.com/docs/en/memory).
- Because the order is broadest→most-specific, **later (more specific) files do not delete earlier rules** — they sit *after* them in the same user message. A project file cannot "turn off" a managed or user rule; it can only add or contradict, and contradictions degrade adherence (see [adherence-and-anti-patterns.md](adherence-and-anti-patterns.md)).
- **Audit move:** managed policy is invisible to the project but always present. If org rules seem to be silently shaping behavior, check the managed path / `claudeMd` key; run `/memory` to see every file actually loaded.

## 2. How files load

**Upward walk.** Discovery starts in cwd and **walks UP the directory tree** toward (but stopping above) root `/`, collecting every `CLAUDE.md` it finds; all are loaded at launch (source: https://code.claude.com/docs/en/memory). This is why launching Claude from a deep subdirectory of a monorepo can pull in several files — starting-directory discipline matters (see [large-codebase-patterns.md](large-codebase-patterns.md)).

**Concatenation order.** Found files are injected **root → cwd** (highest ancestor first, cwd-closest last), nested under the broadest→specific tier order above (source: https://code.claude.com/docs/en/memory). Nothing is merged or deduplicated — it is literal concatenation.

**On-demand subdirectory loading.** A `CLAUDE.md`/`CLAUDE.local.md` *below* cwd is **not** loaded at launch; it loads only when Claude reads a file in that subtree (source: https://code.claude.com/docs/en/memory). This is the one piece of CLAUDE.md that behaves lazily — use it to keep per-package rules out of the always-loaded budget.

**HTML-comment stripping.** Block-level HTML comments (`<!-- ... -->`) are **stripped before injection**, so they cost zero context. Use them for maintainer-only notes (ownership, last-reviewed date, rationale you don't want the model to read) (source: https://code.claude.com/docs/en/memory).

**Live-audit commands:** `/memory` (which files loaded + edit them), `/context` (token budget breakdown — see [context-and-token-economics.md](context-and-token-economics.md)), `/status`, `/doctor`. The `InstructionsLoaded` hook logs exactly which instruction files were ingested for a session — wire it up to verify load order programmatically (source: https://code.claude.com/docs/en/hooks).

## 3. `@path` imports

`@path` syntax pulls another file's contents **inline at launch**, in place of the `@path` token (source: https://code.claude.com/docs/en/memory).

| Property | Behavior |
|---|---|
| Expansion timing | Expanded **inline at launch**, like an `#include` — the imported text becomes part of the same user message (source: https://code.claude.com/docs/en/memory). |
| Recursion limit | Imports may be nested; **max 4 hops** deep (source: https://code.claude.com/docs/en/memory). |
| Path resolution | Both **relative** (`@./docs/api.md`, `@../shared.md`) and **absolute** (`@/abs/path.md`) paths work; `@~/.claude/my-prefs.md` imports from home (source: https://code.claude.com/docs/en/memory). |
| Code-block safety | `@path` inside spans/code fences is **not** evaluated, so it won't trigger spurious imports (source: https://code.claude.com/docs/en/memory). |
| Approval | Importing files **outside the repo tree** prompts an approval dialog before the contents are read in (a security boundary) (source: https://code.claude.com/docs/en/memory). |

- **Myth to kill: imports do NOT save context.** `@path` is expanded inline at launch, so every byte of every imported file counts against the always-loaded budget exactly as if it were pasted in. Imports buy *organization* and reuse, **not** a smaller context footprint — there is no lazy/deferred import. For the budget math and what *does* defer (skills, path-scoped rules, subdirectory CLAUDE.md), see [context-and-token-economics.md](context-and-token-economics.md).
- **Audit move:** a thin-looking root `CLAUDE.md` that `@`-imports five docs is *not* lean — measure the expanded total with `/context`, not the line count of the root file.

## 4. AGENTS.md interop

- **Claude Code reads `CLAUDE.md`, not `AGENTS.md`.** There is no native AGENTS.md ingestion at runtime; the practical bridge is a `CLAUDE.md` whose body is `@AGENTS.md` (or a symlink `CLAUDE.md → AGENTS.md`), keeping a single source of truth for cross-tool repos (source: https://agents.md/).
- **AGENTS.md is now a formal open standard**, not just a convention: it was contributed by OpenAI to the Linux Foundation's **Agentic AI Foundation (AAIF)**, announced 2025-12-09, with Anthropic a co-founder (via MCP) (source: https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation). It is plain Markdown with **no required fields**, used by **60k+ open-source projects** across Codex, Cursor, Jules, Copilot, Devin, Factory, Aider, Zed, Warp and 20+ tools (community count) (source: https://agents.md/).
- **`/init` ingests existing manifests.** When initializing, `/init` reads in pre-existing `AGENTS.md`, `.cursorrules`, and `.windsurfrules` to seed the new `CLAUDE.md` — so migrating from another tool does not start from scratch (source: https://medium.com/@AdithyaGiridharan/claude-codes-init-command-is-getting-a-brain-transplant-e0c389ec0c6b).
- **Audit move:** if a repo has both `AGENTS.md` and a divergent hand-written `CLAUDE.md`, flag the drift. The maintainable shape is one canonical file plus a bridge (`@AGENTS.md` or symlink), not two instruction files that silently disagree.

## 5. What to INCLUDE vs EXCLUDE (the official table)

The governing rule is literal: *"For each line, ask: 'Would removing this cause Claude to make mistakes?' If not, cut it. Bloated CLAUDE.md files cause Claude to ignore your actual instructions!"* (source: https://code.claude.com/docs/en/best-practices).

| INCLUDE (non-inferable, behavior-shifting) | EXCLUDE (inferable, stale, or self-evident) |
|---|---|
| Non-guessable bash commands | Anything inferable by reading the code |
| Code-style rules that **differ from defaults** | Standard conventions Claude already knows |
| Test runners / how to run tests | Detailed API docs (link instead) |
| Repo etiquette (branch naming, merge vs rebase) | Frequently-changing information |
| Project-specific architectural decisions | Long tutorials |
| Environment quirks / setup gotchas | File-by-file codebase descriptions |
| Non-obvious gotchas | Self-evident advice ("write clean code") |

(source: https://code.claude.com/docs/en/best-practices)

- **Tooling/commands are the highest-adherence content.** A study found that naming the `uv` package manager in the context file made agents use it **~160× more often** (community) — concrete commands and tool names are exactly what CLAUDE.md is *for* (source: https://thomas-wiegold.com/blog/claude-md-helpful-or-expensive-noise/).
- **Anchor broadly:** CLAUDE.md loads every session, so only include what applies broadly; sometimes-relevant domain knowledge or workflows belong in **skills** (loaded on demand), not in the always-loaded file (source: https://code.claude.com/docs/en/best-practices).
- **No snippets, no pasted docs.** Embedding code snippets or full API docs is an anti-pattern: they go stale and inflate the always-loaded budget. Use **`file:line` references** and links and let the model read source on demand (source: https://www.humanlayer.dev/blog/writing-a-good-claude-md).

## 6. Authoring for adherence

Anthropic's own prompt-engineering guidance and the literalism of current Opus models converge on five levers. These shape *how a kept line is phrased*, not whether it is kept (that's §5).

| Lever | Rule | Why |
|---|---|---|
| **Positive framing** | Tell Claude *what to do*, not what not to do — "Use named exports (tree-shaking depends on it)" beats "Do NOT use default exports" (source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices). | Negation resolves poorly; the "pink elephant" anecdote — a file saying "NEVER create duplicate files" still produced `file-fixed.py` — is the canonical failure (community) (source: https://eval.16x.engineer/blog/the-pink-elephant-negative-instructions-llms-effectiveness-analysis). |
| **Explain the why** | Attach motivation to each rule (WHY/WHAT/HOW); "explaining why such behavior is important … helps Claude better understand your goals" (source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices). | Context makes responses more on-target than a bare imperative. |
| **State scope explicitly** | Spell out breadth — "apply to every component, not just new ones." | **Opus 4.8 interprets instructions literally and does not silently generalize** from one item to another; under-scoped rules under-apply, especially at lower reasoning effort (source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices). |
| **Examples beat prose** | Show one positive example of the desired behavior. | "Positive examples … tend to be more effective than negative examples or instructions that tell the model what not to do" (source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices). |
| **Emphasis is a SCARCE signal** | Use `IMPORTANT`/`YOU MUST` on the *few* genuinely critical, frequently-violated rules only — never peppered throughout. | Anthropic endorses emphasis as a tuning lever (source: https://code.claude.com/docs/en/best-practices), but the same page lists "the over-specified CLAUDE.md" as a top failure pattern; over-emphasis is self-defeating because everything-is-important means nothing is. |

- **Specificity is safety:** a narrow named prohibition beats a vague category — "do not touch `middleware.ts` without asking" outperforms "don't change auth"; pair a short autonomous-actions list with a short must-ask list rather than a wall of NEVERs (community) (source: https://blink.new/blog/claude-md-best-practices).
- **Primacy/recency placement:** a community pattern duplicates the 2–3 most critical rules at the file's **top and bottom** to exploit primacy/recency, rather than scattering emphasis (source: https://dev.to/docat0209/5-patterns-that-make-claude-code-actually-follow-your-rules-44dh).
- **Diagnose, don't escalate.** Per the official heuristic: a rule violated *despite being present* usually means the **file is too long** and the rule is lost — not that it needs more `IMPORTANT`; re-asking an answered question means the phrasing is **ambiguous** (source: https://code.claude.com/docs/en/best-practices). Adding emphasis to a persistently-ignored rule is an anti-pattern.
- **Keep it small.** A practitioner heuristic puts a workable root file **under ~60 lines**, on the reasoning that frontier models follow only ~150–200 discrete instructions reliably and Claude Code's own system prompt consumes part of that budget (community estimate — treat as a heuristic, not a measured Anthropic limit) (source: https://www.humanlayer.dev/blog/writing-a-good-claude-md). The official sizing target (`<200 lines`) and the budget rationale live in [context-and-token-economics.md](context-and-token-economics.md).

## 7. In-session memory shortcuts

- **`#` shortcut:** typing a line beginning with `#` during a session appends that memory to the **most relevant CLAUDE.md automatically** — practitioners use it for one-liners like "always use MUI components for new stuff" and it routes to the right file (community) (source: https://www.builder.io/blog/claude-code). Natural-language "add this to CLAUDE.md" works the same way through the memory flow.
- **Caveat:** the `#` shortcut grows the file without the include/exclude test — periodically run `/memory` and prune what `#` accreted, or it silently becomes the bloated file §5 warns against.

## 8. `/init` and `CLAUDE_CODE_NEW_INIT=1`

- `/init` scaffolds an initial `CLAUDE.md` from the codebase (and ingests `AGENTS.md`/`.cursorrules`/`.windsurfrules` per §4).
- **`CLAUDE_CODE_NEW_INIT=1`** enables an interview-driven flow (community-dated to a March 2026 upgrade): it uses the `AskUserQuestion` tool to ask about your workflow, which **recurring tasks could become skills**, and which **unsafe operations should be blocked via hooks** — producing a full scaffold configured from intent rather than file inspection alone (community) (source: https://medium.com/@AdithyaGiridharan/claude-codes-init-command-is-getting-a-brain-transplant-e0c389ec0c6b).
- **Never commit `/init` output verbatim.** An ETH Zurich study (reported by Wiegold) found auto-generated context files *decreased* success and raised cost ~20%, and Claude Code was the only agent where even developer-written files failed to help when generic docs already existed (community) (source: https://thomas-wiegold.com/blog/claude-md-helpful-or-expensive-noise/). Treat `/init` as a **draft**: review against the include/exclude table (§5), strip the file-tree description and self-evident advice, prune to non-inferable facts, then commit.

## Quick audit checklist

- [ ] Root `CLAUDE.md` short (heuristic <60 lines; official `<200`); expanded total (post-`@import`) measured with `/context`, not raw line count.
- [ ] Every line survives "would removing this cause a specific mistake?"; no snippets, no pasted API docs, no file-by-file tree.
- [ ] Rules are positive, scoped, with the *why*; emphasis only on the few critical/often-violated ones.
- [ ] Must-hold rules moved to **hooks/permissions**, not trusted to prose; sometimes-relevant rules in **skills / path-scoped rules**.
- [ ] If `AGENTS.md` exists, it's the single source of truth bridged by `@AGENTS.md`/symlink — no divergent twin.
- [ ] `/init` output reviewed and pruned, not committed verbatim; managed policy checked via `/memory`.

See [choosing-the-right-mechanism.md](choosing-the-right-mechanism.md) for the load-timing / context-cost / guarantee routing rubric, and [audit-rubric.md](audit-rubric.md) for the scored dimensions.

## Sources
1. https://code.claude.com/docs/en/memory — *(official)* memory file locations, load order, upward walk, concatenation order, on-demand subdir loading, HTML-comment stripping, `CLAUDE.local.md` deprecation, `@path` imports (inline-at-launch, 4-hop, path resolution, approval dialog), context-not-config.
2. https://code.claude.com/docs/en/best-practices — *(official)* the "would removing this line" test, include/exclude table, anchor-broadly, emphasis-as-tuning-lever vs over-specified failure pattern, diagnose-don't-escalate heuristic, hooks-are-deterministic.
3. https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices — *(official)* positive framing, explain-the-why, state-scope-explicitly (Opus 4.8 literalism), positive examples beat negative instructions.
4. https://code.claude.com/docs/en/admin-setup — *(official)* managed-policy paths and `claudeMd` key; cannot be excluded.
5. https://code.claude.com/docs/en/hooks — *(official)* `InstructionsLoaded` hook logs which instruction files loaded.
6. https://agents.md/ — *(community)* AGENTS.md spec, 60k+ projects / 20+ tools, plain-Markdown/no-required-fields, the `@AGENTS.md`/symlink bridge.
7. https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation — *(official)* AAIF formation, AGENTS.md + MCP as project contributions, 2025-12-09.
8. https://medium.com/@AdithyaGiridharan/claude-codes-init-command-is-getting-a-brain-transplant-e0c389ec0c6b — *(community)* `CLAUDE_CODE_NEW_INIT=1` interview flow; `/init` ingesting AGENTS.md/.cursorrules/.windsurfrules.
9. https://thomas-wiegold.com/blog/claude-md-helpful-or-expensive-noise/ — *(community)* ETH Zurich study coverage (auto files net-negative, ~20% cost), `uv` 160× tooling-adherence figure.
10. https://www.humanlayer.dev/blog/writing-a-good-claude-md — *(community)* tiny-root-file heuristic, `file:line` over snippets, ~150–200 instruction-budget framing.
11. https://eval.16x.engineer/blog/the-pink-elephant-negative-instructions-llms-effectiveness-analysis — *(community)* pink-elephant negative-instruction anecdote.
12. https://blink.new/blog/claude-md-best-practices — *(community)* specificity-as-safety (named prohibition beats vague category).
13. https://dev.to/docat0209/5-patterns-that-make-claude-code-actually-follow-your-rules-44dh — *(community)* primacy/recency top-and-bottom placement of critical rules.
14. https://www.builder.io/blog/claude-code — *(community)* in-session `#` add-to-memory shortcut auto-routing to the most relevant file.

## Confidence & gaps
- **Verified official (safe to state as fact):** load locations and broadest→specific concatenation order; the upward directory walk and root→cwd ordering; on-demand subdirectory loading; HTML-comment stripping; `CLAUDE.local.md` deprecation; `@path` inline-at-launch expansion, 4-hop limit, relative/absolute resolution, code-block non-evaluation, and the outside-repo approval dialog; managed-policy paths and non-excludability; the include/exclude table and the "would removing this line" test; emphasis-as-tuning-lever and the over-specified failure pattern; positive framing / explain-the-why / state-scope / positive-examples; Opus 4.8 literalism; the `InstructionsLoaded` hook (sources 1–5).
- **Community / inference (do not assert as Anthropic facts):** the ETH Zurich numbers (~20% cost, net-negative auto files, the 2.7%/4% lifts) and the `uv` 160× figure are preprint/secondary and partly anecdotal — verify before quoting as hard numbers; the ~150–200-instruction budget and "system prompt eats ~50" are estimates, not a published benchmark; the pink-elephant "halved violations" number was unsupported and is **not** cited here; the `#`-shortcut routing and AGENTS.md adoption count (60k+/20+ tools) are community-reported.
- **Version-sensitive / may be stale:** `CLAUDE_CODE_NEW_INIT=1`'s "March 2026" framing is community-dated, not from a fetched official changelog — the env var and multi-phase flow are confirmed but the date is not. The `@path` semantics, 4-hop limit, and `CLAUDE.local.md` deprecation are current as of 2026-06-04 and could change; re-verify against https://code.claude.com/docs/en/memory and the changelog. Whether the positive-framing gains were measured *specifically* on Opus 4.8 (vs 4.6/4.7) is not established; the literalism note implies scope-explicitness matters more than ever, but no A/B was found.

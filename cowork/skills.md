# Agent Skills: Best Practices and Architecture Guide

*A source-backed deep-dive for skill authors and workflow builders, with a focus on Claude Cowork and the shared Skills system across Claude Code, claude.ai, the Claude API, and the Agent SDK.*

---

## 1. What a Skill is

A **Skill** is an organized folder of instructions, scripts, and resources that an agent discovers and loads dynamically to perform better at a specific task. Anthropic describes it as functioning like onboarding documentation for a new team member: it makes a general-purpose agent into a specialist without a custom-built integration (source: https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills).

At minimum, a Skill is a directory containing a single required file, `SKILL.md`, which carries YAML frontmatter (metadata) followed by a Markdown body (the instructions). It may bundle additional files — reference docs, scripts, templates, assets — that are loaded only when needed (source: https://agentskills.io/specification; source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview).

Skills are now an **open standard** (originally developed by Anthropic, released publicly on 2025-12-18) adopted by a large ecosystem of agent products beyond Anthropic — Cursor, GitHub Copilot/VS Code, Gemini CLI, OpenAI Codex, Goose, and many others — so a Skill authored once is portable across compatible agents (source: https://agentskills.io/home; source: https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills). Skills were first announced on 2025-10-16 (source: https://claude.com/blog/skills).

### 1.1 Directory structure

The canonical layout from the open standard (source: https://agentskills.io/specification):

```
skill-name/
├── SKILL.md          # Required: metadata + instructions
├── scripts/          # Optional: executable code agents can run
├── references/       # Optional: documentation loaded on demand
├── assets/           # Optional: templates, images, data files
└── ...               # Any additional files or directories
```

- `scripts/` — executable code (Python, Bash, JavaScript depending on the agent). Self-contained or with documented dependencies, helpful error messages, graceful edge-case handling (source: https://agentskills.io/specification).
- `references/` — additional docs (e.g. `REFERENCE.md`, `FORMS.md`, domain files like `finance.md`) loaded only when relevant. Keep individual files focused so loading costs little context (source: https://agentskills.io/specification).
- `assets/` — static resources used in output: document/config templates, images, lookup tables, schemas (source: https://agentskills.io/specification; source: https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md).

### 1.2 SKILL.md YAML frontmatter — the authoritative schema

This is the single most load-bearing detail, so here is the exact schema from the open standard specification (source: https://agentskills.io/specification):

| Field | Required | Constraints |
|---|---|---|
| `name` | **Yes** | Max 64 chars. Lowercase letters, numbers, hyphens only. Must not start/end with a hyphen. No consecutive hyphens (`--`). **Must match the parent directory name.** |
| `description` | **Yes** | Max 1024 chars. Non-empty. Describes *what the skill does and when to use it*. |
| `license` | No | License name or reference to a bundled license file (keep short, e.g. `Apache-2.0` or `Proprietary. LICENSE.txt has complete terms`). |
| `compatibility` | No | Max 500 chars. Environment requirements: intended product, system packages, network access (e.g. `Requires git, docker, jq, and access to the internet`). Most skills do not need this. |
| `metadata` | No | Arbitrary string→string key-value map for client-specific properties not defined by the spec (recommended: namespaced keys to avoid collisions; e.g. `author`, `version`). |
| `allowed-tools` | No | **Experimental.** Space-separated string of pre-approved tools the skill may use (e.g. `Bash(git:*) Bash(jq:*) Read`). Support varies by implementation. |

> **Important nuance on `version`:** There is **no top-level `version` frontmatter field** in the open standard. Versioning is expressed informally inside the free-form `metadata` map (the spec's own example uses `metadata: { author: example-org, version: "1.0" }`) (source: https://agentskills.io/specification). Some third-party help content references a `dependencies` field, but that is **not** in the canonical spec — treat `compatibility` (spec) for environment needs and `metadata` for versioning/authorship as authoritative (community claim flagged: https://support.claude.com/en/articles/12512198-how-to-create-custom-skills).

**Minimal valid SKILL.md** (verified against the spec, source: https://agentskills.io/specification):

```markdown
---
name: pdf-processing
description: A description of what this skill does and when to use it.
---

# PDF Processing

## Instructions
[Clear, step-by-step guidance for Claude to follow]

## Examples
[Concrete input/output examples]
```

**With optional fields** (verbatim from spec, source: https://agentskills.io/specification):

```markdown
---
name: pdf-processing
description: Extract PDF text, fill forms, merge files. Use when handling PDFs.
license: Apache-2.0
metadata:
  author: example-org
  version: "1.0"
---
```

> Note: Anthropic's own platform docs and the engineering blog emphasize only `name` + `description` as "the two required fields." The fuller field list (`license`, `compatibility`, `metadata`, `allowed-tools`) comes from the formal open-standard specification at agentskills.io and from Claude Code's extended frontmatter (§7) (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview; source: https://agentskills.io/specification).

The `name`/`description` validation rules are mirrored in Anthropic's platform docs: `name` ≤ 64 chars, lowercase/numbers/hyphens, no XML tags, **cannot contain the reserved words "anthropic" or "claude"**; `description` non-empty, ≤ 1024 chars, no XML tags (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview; source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).

---

## 2. The progressive disclosure model

Progressive disclosure is the **core design principle** that makes Skills scalable. Claude loads information in stages rather than reading an entire Skill into context up front. Anthropic likens it to a well-organized manual: table of contents → specific chapters → detailed appendix (source: https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills).

### 2.1 The three loading levels and their token cost

From the official overview's own table (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview):

| Level | When loaded | Token cost | Content |
|---|---|---|---|
| **L1: Metadata** | Always, at startup | **~100 tokens per Skill** | `name` + `description` from YAML frontmatter, injected into the system prompt |
| **L2: Instructions** | When the Skill is triggered | **Under ~5k tokens** (keep SKILL.md body < 500 lines) | The SKILL.md Markdown body |
| **L3+: Resources** | As needed | **Effectively unlimited** | Bundled files read via bash; scripts executed without loading their source into context |

The open standard states the same in slightly different units: metadata ~100 tokens loaded at startup for *all* skills; instructions recommended < 5,000 tokens; resources loaded only when required (source: https://agentskills.io/specification).

### 2.2 Why this matters — the context-window economics

- **Many Skills, near-zero idle cost.** Because only metadata is preloaded, you can install dozens or hundreds of Skills with negligible context penalty; Claude only knows each one *exists* and *when to use it* (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview).
- **Scripts are token-free to run.** When Claude executes a bundled script via bash, the script's *source code never enters context* — only its stdout/stderr does. This makes pre-written utility scripts dramatically more efficient than having Claude generate equivalent code on the fly (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview).
- **No penalty for comprehensive bundled docs.** A Skill can ship large API references or datasets; unread files consume zero tokens until accessed (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview).
- **The context window is a public good.** Once SKILL.md is loaded, every token competes with conversation history, other Skills' metadata, the system prompt, and the user's actual request — so conciseness in the body still matters even though it is not preloaded (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).

### 2.3 How and when Claude decides to invoke a Skill — the role of `description`

The filesystem mechanics (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview):

1. **Startup:** every Skill's `name` + `description` are loaded into the system prompt.
2. **Match:** when a user request matches a Skill's description, Claude issues a bash command to read `SKILL.md` from the filesystem — only then does L2 enter context.
3. **Drill-down:** if SKILL.md references `FORMS.md` or a schema, Claude reads those too via additional bash calls; referenced scripts are executed (output only).

`description` is the **single primary triggering mechanism.** Anthropic: "Claude uses it to choose the right Skill from potentially 100+ available Skills. Your description must provide enough detail for Claude to know when to select this Skill" (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).

A critical, often-missed behavior documented in Anthropic's own `skill-creator` skill: **Claude only consults Skills for tasks it can't easily handle on its own.** "Simple, one-step queries like 'read this PDF' may not trigger a skill even if the description matches perfectly, because Claude can handle them directly with basic tools. Complex, multi-step, or specialized queries reliably trigger skills when the description matches" (source: https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md). This explains why a well-described Skill can still appear "not to fire" on trivial requests — it is expected.

Also documented: Claude currently has a **tendency to *under*-trigger** Skills. Anthropic's skill-creator explicitly advises making descriptions "a little bit 'pushy'" — instead of *"How to build a simple fast dashboard,"* write *"…Make sure to use this skill whenever the user mentions dashboards, data visualization, internal metrics, or wants to display any kind of company data, even if they don't explicitly ask for a 'dashboard.'"* (source: https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md).

---

## 3. Authoring best practices

All of §3 is sourced from Anthropic's official best-practices guide unless noted (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).

### 3.1 Names and descriptions for discovery

- **Names:** prefer **gerund form** (verb + -ing): `processing-pdfs`, `analyzing-spreadsheets`, `managing-databases`. Acceptable alternatives: noun phrases (`pdf-processing`) or action-oriented (`process-pdfs`). Avoid vague (`helper`, `utils`, `tools`), overly generic (`documents`, `data`), and reserved words (`anthropic-*`, `claude-*`). Keep naming consistent across your library.
- **Descriptions — write in third person, always.** The description is injected into the system prompt; inconsistent point-of-view breaks discovery.
  - Good: `"Processes Excel files and generates reports"`
  - Avoid: `"I can help you process Excel files"` / `"You can use this to process Excel files"`
- **Be specific; include both *what* and *when*, plus trigger keywords.** Canonical good examples:
  - `description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.`
  - `description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.`
- **Anti-examples to never ship:** `Helps with documents`, `Processes data`, `Does stuff with files`.
- Per the skill-creator: *all* "when to use" information belongs in `description`, **not** in the body, because the body isn't visible at trigger time (source: https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md).

### 3.2 Instruction style — "Claude is already very smart"

- **Conciseness test.** For every paragraph ask: "Does Claude really need this? Can I assume Claude knows it? Does it justify its token cost?" The guide's example: a ~50-token concise PDF section beats a ~150-token version that explains what a PDF is.
- **Set appropriate degrees of freedom** — match specificity to task fragility:
  - *High freedom* (prose instructions): multiple valid approaches, context-dependent (e.g. code review).
  - *Medium freedom* (pseudocode / parameterized scripts): a preferred pattern exists, some variation OK.
  - *Low freedom* (exact scripts, no params): fragile/destructive, consistency critical (e.g. `Run exactly this script: python scripts/migrate.py --verify --backup. Do not modify the command`). Analogy: narrow bridge with cliffs vs. open field.
- **Explain the *why*, don't just dictate.** The skill-creator is emphatic: *"If you find yourself writing ALWAYS or NEVER in all caps, or using super rigid structures, that's a yellow flag — if possible, reframe and explain the reasoning so that the model understands why the thing you're asking for is important."* Modern models have good theory of mind and perform better with rationale than rote MUSTs (source: https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md).
- **Use imperative form** in instructions; **consistent terminology** throughout (pick one of "field"/"box"/"element" and stick to it); **concrete examples, not abstract**; provide input/output example pairs where output quality depends on style.
- **Workflows & feedback loops:** for complex multi-step tasks, give Claude a checklist it can copy into its response and tick off. Implement the **validator → fix → repeat** loop ("Only proceed when validation passes"). Use the **plan → validate → execute** pattern for batch/destructive operations (create an intermediate `changes.json`, validate it with a script before applying).
- **Templates:** match strictness to need — `ALWAYS use this exact template` for strict formats vs. "a sensible default… use your best judgment" for flexible ones.

### 3.3 Focused / single-purpose

Keep SKILL.md **under 500 lines** (≈ < 5k tokens). When it grows past that, split content into separate reference files and link them. Organize by domain so irrelevant context never loads (e.g. a BigQuery skill with `reference/finance.md`, `reference/sales.md`, `reference/product.md` — a revenue question loads only finance.md). Keep references **one level deep from SKILL.md**: Claude may only `head -100`-preview files reached through nested reference chains, getting incomplete info. For reference files > 100 lines, add a table of contents at the top so partial previews still reveal scope (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).

### 3.4 Scripts vs. natural-language instructions

Use **code where determinism matters**; use prose where judgment matters (source: https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills). Benefits of bundling utility scripts even when Claude *could* write them: more reliable than generated code, save tokens (source never enters context), save time, ensure consistency (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).

- **Make execution intent explicit:** "Run `analyze_form.py` to extract fields" (execute) vs. "See `analyze_form.py` for the algorithm" (read as reference). Execution is preferred for most utility scripts.
- **Code as documentation:** scripts can serve as both executable tools and reference (source: https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills).
- **Repeated work signal:** if every test run independently rewrites a similar helper, that helper belongs in `scripts/` (source: https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md).

### 3.5 Error handling & idempotence ("solve, don't punt")

- **Handle errors in scripts; don't punt to Claude.** Good: catch `FileNotFoundError`/`PermissionError` and create a default or provide an alternative. Bad: `return open(path).read()` and "let Claude figure it out."
- **No "voodoo constants" (Ousterhout's law):** justify and document every config value with a comment (`REQUEST_TIMEOUT = 30  # accounts for slow connections`), never `TIMEOUT = 47  # Why 47?`. "If you don't know the right value, how will Claude determine it?"
- **Idempotence / safe re-runs:** the design pattern Anthropic promotes for risky operations is **verifiable intermediate outputs** — plan → validate → execute → verify — so a run can be re-attempted from a validated plan without corrupting originals. Use it for batch operations, destructive changes, and high-stakes work. Make validation scripts *verbose* with specific error messages listing valid options.
(All: source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)

### 3.6 Versioning

There is no enforced version mechanism. The pragmatic approach: track a version string in the `metadata` map (`metadata: { version: "1.0" }`) per the open standard (source: https://agentskills.io/specification), and **avoid time-sensitive content** in the body. Instead of "before August 2025 use the old API," keep a "Current method" section plus a collapsed `<details>` "Old patterns" section for historical context that won't rot (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices). For distribution, version control is the real mechanism: commit project Skills to git; the skill-creator preserves the original `name` when updating an existing skill and copies it to `/tmp/` before editing (source: https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md).

### 3.7 Testing & evaluation — build evals *first*

Anthropic's headline practice: **create evaluations BEFORE writing extensive documentation** so the Skill solves real problems, not imagined ones. The loop (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices):

1. Run Claude on representative tasks *without* the Skill; document concrete failures.
2. Build ≥ 3 evaluation scenarios targeting those gaps.
3. Establish a baseline (no Skill).
4. Write minimal instructions to pass the evals.
5. Iterate against the baseline.

Eval record schema (verbatim):

```json
{
  "skills": ["pdf-processing"],
  "query": "Extract all text from this PDF file and save it to output.txt",
  "files": ["test-files/document.pdf"],
  "expected_behavior": [
    "Successfully reads the PDF using an appropriate library or tool",
    "Extracts text from all pages without missing any",
    "Saves the extracted text to output.txt in a clear, readable format"
  ]
}
```

There is **no built-in eval runner**; you build your own harness (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices). Anthropic's `skill-creator` skill, however, ships an opinionated, automated eval pipeline you can drive conversationally (§6).

**Test with every model you'll use.** Skills are additions to a model; what's perfect for Opus may underspecify for Haiku. Validate on Haiku/Sonnet/Opus (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).

### 3.8 Iterating with Claude's help — the "Claude A / Claude B" method

Anthropic's recommended development process uses two roles (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices):

- **Claude A** — the expert that helps you *write/refine* the SKILL.md (Claude understands the format natively; no special prompt or "skill-writing skill" required).
- **Claude B** — a fresh instance that *uses* the Skill on real tasks.

Loop: complete a task with Claude A using normal prompting → notice repeated context you provide → ask Claude A to capture it as a Skill → review for conciseness ("remove the explanation of win rate; Claude knows that") → improve information architecture (move schemas to reference files) → test with Claude B on real work → observe failures → bring specifics back to Claude A. Watch how Claude B navigates: unexpected exploration order (structure isn't intuitive), missed reference links (make them more prominent), repeatedly re-read file (promote it into SKILL.md), never-read file (cut it or signal it better). Gather teammate feedback to cover blind spots.

### 3.9 Effective-Skill checklist (Anthropic's, condensed)

Core: description is specific + has key terms + states what *and* when; SKILL.md < 500 lines; extra detail in separate files; no time-sensitive info; consistent terminology; concrete examples; references one level deep; workflows have clear steps. Code: scripts solve not punt; explicit error handling; no voodoo constants; required packages listed and verified available; forward-slash paths only; validation/feedback loops for critical ops. Testing: ≥ 3 evals; tested on Haiku/Sonnet/Opus; tested with real usage; team feedback incorporated (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).

---

## 4. Anti-patterns and common mistakes

Sourced from the best-practices guide and skill-creator unless noted:

- **Bloated SKILL.md.** Over-explaining things Claude already knows; > 500-line bodies. Fix with progressive disclosure (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).
- **Vague descriptions.** `"Helps with documents"` — Claude can't discover it. Also: descriptions written in first/second person break system-prompt injection (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).
- **Putting "when to use" in the body** instead of `description` — the body isn't visible at trigger time (source: https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md).
- **Deeply nested references** (SKILL.md → advanced.md → details.md) → partial reads, lost info. Keep one level deep (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).
- **Offering too many options.** "Use pypdf, or pdfplumber, or PyMuPDF, or…" confuses Claude — give a default with one escape hatch (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).
- **Windows-style paths** (`scripts\helper.py`) break on Unix — always forward slashes (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).
- **Assuming packages/tools are installed**; assuming network access (no network on the Claude API runtime). Declare dependencies; verify availability (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices; source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview).
- **Unqualified MCP tool names.** Always use `ServerName:tool_name` (e.g. `GitHub:create_issue`); bare names cause "tool not found" when multiple MCP servers exist (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).
- **Time-sensitive content** that rots — use an "Old patterns" `<details>` section (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).
- **Rigid ALL-CAPS ALWAYS/NEVER** instead of explained reasoning — a "yellow flag" per skill-creator (source: https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md).
- **Overfitting to test cases.** Generalize from feedback so the Skill works across many prompts, not just the evals (source: https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md).
- **Overlapping / over-scoped Skills.** The whole authoring philosophy is focused, single-purpose Skills with non-overlapping descriptions; multi-domain content should be split into reference files, and near-miss / adjacent-domain "should-not-trigger" cases should be tested explicitly via description optimization (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices; source: https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md).
- **Secrets in Skills.** Skills are treated like installable software. Never bundle credentials; a malicious or compromised Skill can direct Claude to exfiltrate data or misuse tools. Skills that fetch from external URLs are especially risky (fetched content may carry injected instructions). Audit every bundled file — SKILL.md, scripts, images — for unexpected network calls or out-of-scope file access. Only use Skills you authored or got from Anthropic (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview). The skill-creator's **"Principle of Lack of Surprise"**: *"Skills must not contain malware, exploit code, or any content that could compromise system security. A skill's contents should not surprise the user in their intent if described."* (source: https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md).
- **Auto-triggering destructive workflows.** For deploy/commit/send-message, prevent automatic invocation (use `disable-model-invocation: true` in Claude Code; in claude.ai/Cowork, manage via enable/disable and explicit invocation) (source: https://code.claude.com/docs/en/skills).

---

## 5. Skill scope, distribution, and governance

### 5.1 Sharing model differs by surface (this is the #1 operational gotcha)

Skills **do not sync across surfaces**; you manage and upload them separately per surface (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview):

| Surface | Where Skills live | Sharing scope |
|---|---|---|
| **claude.ai (Cowork & Chat)** | Uploaded as a `.zip` via Settings → (Customize →) Skills | **Per-individual user.** Custom Skills are private to your account; not org-managed *unless an admin provisions them* (see §5.3) (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview; source: https://support.claude.com/en/articles/12512180-use-skills-in-claude) |
| **Claude API** | Uploaded via `/v1/skills`; reference by `skill_id` in the `container` param with the code-execution tool | **Workspace-wide** — all workspace members can access (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) |
| **Claude Code** | Filesystem dirs: personal `~/.claude/skills/`, project `.claude/skills/`, enterprise via managed settings, plugin `<plugin>/skills/` | Personal, project (commit to git), plugin, or org-managed (source: https://code.claude.com/docs/en/skills) |
| **Claude Agent SDK** | Inherits Claude Code's filesystem model (Agent Skills support is built in) (source: https://claude.com/blog/skills) |

API beta headers (current): `code-execution-2025-08-25`, `skills-2025-10-02`, `files-api-2025-04-14` (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview).

### 5.2 Personal vs. project vs. enterprise (Claude Code precedence)

Claude Code's precedence when names collide: **enterprise > personal > project**; plugin Skills use a `plugin-name:skill-name` namespace so they never conflict (source: https://code.claude.com/docs/en/skills).

### 5.3 Org/Team/Enterprise provisioning and the Skills directory

- **Only Owners** can add/remove org-wide Skills. Requires Team/Enterprise plan, with **Code execution and file creation** and **Skills** toggles enabled in Organization settings (source: https://support.claude.com/en/articles/13119606-provision-and-manage-skills-for-your-organization).
- **Provisioning steps:** Organization settings → Skills → Organization skills → **+ Add** → select a `.zip` containing `SKILL.md` → "immediately provisioned to all users." Admin-provisioned Skills are **enabled by default for everyone**, but individual users may toggle them off; users cannot delete them (source: https://support.claude.com/en/articles/13119606-provision-and-manage-skills-for-your-organization; source: https://support.claude.com/en/articles/12512180-use-skills-in-claude).
- **Member sharing controls (both default OFF, enabled in Organization settings → Skills):** (1) share a Skill with specific colleagues (appears in their "Shared with you"), and (2) publish a Skill to the **organization directory** for anyone to find/install. **There is no approval workflow for org-wide sharing.** Skill-sharing events are captured in the **audit log and Compliance API as `role_assignment` events** (source: https://support.claude.com/en/articles/13119606-provision-and-manage-skills-for-your-organization).
- **Skills directory / partner Skills** (announced 2025-12-18): a directory at claude.com/connectors featuring partner-built Skills/integrations — Atlassian (Jira/Confluence), Canva, Figma, Cloudflare, Vercel, Sentry, Zapier, Notion, and more (source: https://claude.com/blog/organization-skills-and-directory; source: https://github.com/anthropics/skills/blob/main/README.md).

### 5.4 Plugins / marketplace relationship

- In **Claude Code**, Skills are distributed in three scopes: project (commit `.claude/skills/`), **plugins** (a `skills/` dir inside a plugin), or org-managed settings. The public Anthropic repo doubles as a plugin marketplace: `/plugin marketplace add anthropics/skills`, then install `document-skills` or `example-skills` plugins (source: https://code.claude.com/docs/en/skills; source: https://github.com/anthropics/skills/blob/main/README.md).
- **In Cowork specifically, the packaging unit above a Skill is a "plugin."** Anthropic's Cowork docs: *"Plugins customize how Claude works for your role, team, and company in Cowork. Each one bundles skills, connectors, and sub-agents into a single package."* (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork). Conceptually: a **Skill** = one repeatable procedure; a **plugin** = a curated bundle of Skills + connectors (MCP) + sub-agents shipped together.

---

## 6. Cowork-specific guidance (for the power user)

### 6.1 What Cowork is and how Skills fit

Claude Cowork brings the **same agentic architecture that powers Claude Code into Claude Desktop, without the terminal**, for knowledge work beyond coding. You describe an outcome, step away, and return to finished work. Claude analyzes the request, plans, breaks work into subtasks, runs code/shell commands **inside an isolated VM on your computer**, can coordinate parallel workstreams, and delivers outputs to your filesystem (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork). Because Cowork *is* Claude Code's engine, it supports **Custom Skills** the same filesystem way (and inherits the Agent Skills standard), unlike claude.ai Chat which uses the zip-upload model (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview; community corroboration: https://sherlock.xyz/post/how-to-write-skills-for-claude-code-and-cowork).

### 6.2 How a power user creates/manages Skills in Cowork

- **The easy path — `/skill-creator`.** Type `/skill-creator` in a Cowork session and describe the workflow; Claude interviews you about the use case, writes a draft, generates test prompts, runs them, and helps you evaluate and iterate. No coding required for instruction-only Skills; you can attach executable scripts for advanced ones (source: https://support.claude.com/en/articles/12512176-what-are-skills; community step detail: https://sherlock.xyz/post/how-to-write-skills-for-claude-code-and-cowork).
- **Teaching by doing.** Anthropic's tutorial frames the non-developer flow as: identify a repeatable task where you've refined your approach → describe it to Claude → Claude builds and structures a properly formatted Skill file. When a request matches the description, Claude loads the Skill automatically (source: https://claude.com/resources/tutorials/teach-claude-your-way-of-working-using-skills).
- **Managing Skills.** All Skills appear in **Customize → Skills** (in Cowork, click "Customize" in the left sidebar, then **+** to browse the organization directory / install). You see Anthropic-maintained Skills (Excel/Word/PowerPoint/PDF), your custom Skills, org-provisioned Skills, and partner Skills, each with an on/off toggle (source: https://support.claude.com/en/articles/12512180-use-skills-in-claude; source: https://support.claude.com/en/articles/12512176-what-are-skills).
- **Storage:** Cowork follows Claude Code's filesystem convention — personal `~/.claude/skills/<name>/SKILL.md` (all projects) and project `.claude/skills/<name>/SKILL.md` (that folder/repo only). *(This location is documented authoritatively for Claude Code; Cowork's parity is stated by Anthropic — "same architecture as Claude Code" — and detailed explicitly only by community sources. Flagged in §10.)* (source: https://code.claude.com/docs/en/skills; community: https://sherlock.xyz/post/how-to-write-skills-for-claude-code-and-cowork).

### 6.3 Cowork sandbox, tools, and permissions

- Shell commands and code Claude writes run **inside an isolated VM, separate from your main OS**, but Claude makes *real changes* to files in folders you've connected (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).
- **Controlled file and network access:** Claude reads/writes only in connected folders; network access follows configured **egress settings** (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork). Per the platform docs, claude.ai-surface Skills have **varying network access** (full/partial/none) depending on user/admin settings — unlike the Claude API runtime, which has *no* network and no runtime package installs (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview). This means a Cowork Skill *can* call out / install packages when egress allows, but you should still declare dependencies for portability.
- **Connectors (MCP)** are governed by permissions ("which MCPs you connect, and how often they ask for permission") (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).

### 6.4 Skills vs. Cowork "workflows", folder instructions, and schedules

There is no separate "workflows" object distinct from Skills in Cowork's documentation — **Anthropic uses "skills" as the name for the reusable, repeatable workflows** ("automate recurring tasks with reusable workflows called skills") (source from search of https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork). Distinguish three Cowork mechanisms:

- **Skills** = repeatable, model-discoverable procedures (auto-loaded when relevant). Best for "the way we do X."
- **Folder instructions** = project-specific context attached when you select a local folder; Claude can update them itself during a session. Best for per-project standing context (analogous to a scoped CLAUDE.md) (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).
- **Schedules** (`/schedule`) = run a task automatically or on demand; only run while the computer is awake and Claude Desktop is open. Best for recurring time-based execution; pair a Schedule with a Skill so the scheduled run executes a consistent procedure (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).

### 6.5 The skill-creator's Cowork-specific behavior (for advanced use)

Anthropic's `skill-creator` skill has explicit Cowork branches (source: https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md):

- Cowork **has subagents**, so the full draft → test → benchmark → iterate loop works (unlike claude.ai Chat, which runs one test at a time, skips baselines/benchmarking).
- **No browser/display:** generate the eval review as a static file (`--static <output_path>`) instead of a server; **generate the eval viewer *before* evaluating inputs yourself** to get results to the human ASAP.
- Review **feedback downloads as `feedback.json`** — read it after requesting access.
- Packaging works via Python + filesystem: `python -m scripts.package_skill <path/to/skill-folder>`. When updating an existing Skill, **preserve the original name** and copy to `/tmp/` first, then package from the copy.
- The full skill-creator loop: Decide → draft → run claude-with-the-skill on 2–3 realistic test prompts (saved to `evals/evals.json`) → spawn with-skill *and* baseline runs → draft objective assertions → grade → `python -m scripts.aggregate_benchmark` → analyst pass → static viewer → read feedback (empty = fine) → improve (generalize, keep lean, explain *why*) → repeat. It also offers a **description-optimization** sub-flow: generate ~20 trigger queries (≈10 should-trigger including non-obvious phrasings, ≈10 should-not-trigger near-misses/adjacent domains), run `scripts.run_loop` (60/40 train/held-out split, 3 runs/query, ≤5 iterations, picks `best_description` by *test* score to avoid overfitting), then write `best_description` back into the frontmatter (source: https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md).

---

## 7. Claude Code's extended frontmatter (relevant if you also use Claude Code/SDK)

Claude Code follows the open standard but adds extra optional frontmatter fields beyond the spec. **All fields are optional; only `description` is recommended.** Key additions (source: https://code.claude.com/docs/en/skills):

| Field | Purpose |
|---|---|
| `name` | Display name; defaults to directory name. |
| `description` | What it does + when to use; combined with `when_to_use` and **truncated at 1,536 chars** in the skill listing. Put the key use case first. |
| `when_to_use` | Extra trigger phrases / example requests; appended to description, counts toward the 1,536-char cap. |
| `disable-model-invocation: true` | Only the user can invoke (`/name`); removes it from Claude's context. Use for side-effecting workflows (`/deploy`, `/commit`, `/send-slack-message`). |
| `user-invocable: false` | Only Claude can invoke; hides from the `/` menu. Use for background knowledge that isn't an actionable command. |
| `allowed-tools` | Pre-approves tools while the skill is active (does not restrict others). For project skills, takes effect after accepting the workspace trust dialog — review before trusting. |
| `argument-hint`, `arguments` | Autocomplete hint and named positional args for `$name` / `$ARGUMENTS` / `$0` substitution. |
| `model`, `effort` | Override model/effort for the skill's turn. |
| `context: fork`, `agent` | Run the skill as a forked subagent (skill body becomes the subagent's task prompt). |
| `hooks`, `paths`, `shell` | Lifecycle hooks; glob-scoped activation; bash vs powershell for inline `` !`cmd` ``. |

Two Claude Code mechanics worth knowing for any heavy author:

- **Custom slash commands have merged into Skills.** `.claude/commands/deploy.md` and `.claude/skills/deploy/SKILL.md` both create `/deploy`; old command files still work; if a skill and command share a name, the skill wins (source: https://code.claude.com/docs/en/skills).
- **Skill content lifecycle:** once invoked, the rendered SKILL.md enters the conversation **once and stays for the rest of the session** (Claude Code does not re-read it per turn) — so write standing instructions, not one-time steps; every line is a recurring token cost. Auto-compaction re-attaches the most recent invocation of each skill (first ~5k tokens each, 25k combined budget, most-recent first) (source: https://code.claude.com/docs/en/skills). Skill descriptions consume a listing budget (1% of context by default; raise via `skillListingBudgetFraction`); overflow drops least-used descriptions first — diagnose with `/doctor` (source: https://code.claude.com/docs/en/skills).

---

## 8. Skills vs. the alternatives — decision guide

| Use this | When | Why |
|---|---|---|
| **Skill** | A repeatable, multi-step procedure or domain expertise Claude should apply automatically (or via `/name`) across conversations; "the way we do X." | Loads on demand via progressive disclosure → reusable, low context cost, model-discoverable, portable across surfaces (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview). |
| **Project / folder instructions (CLAUDE.md, Cowork folder instructions)** | Persistent, always-on context that should shape *every* response for a project (conventions, accumulated context). | Always in context — not gated by a trigger. Use Skills instead for long reference material that should cost nothing until needed (source: https://code.claude.com/docs/en/skills; source: https://claude.com/resources/tutorials/teach-claude-your-way-of-working-using-skills). |
| **Custom instructions / preferences** | Universal style preferences about how Claude interacts. | Global tone, not task-specific procedure (source: https://claude.com/resources/tutorials/teach-claude-your-way-of-working-using-skills). |
| **Slash command (Claude Code)** | You mainly want to *trigger the same prompt on demand* with explicit control and no auto-firing. | Now a Skill variant with `disable-model-invocation: true`; explicit invocation, no model decision (source: https://code.claude.com/docs/en/skills). |
| **MCP / connectors** | You need *live external data or actions* (databases, APIs, Jira, Slack, filesystems beyond the working dir). | MCP is the transport/integration layer; Skills are the instructions. They compose — a Skill can instruct Claude to call `ServerName:tool` (source: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices; community framing: https://www.morphllm.com/claude-code-plugins-vs-skills). |
| **Memory / projects** | Context that accumulates over time within an ongoing effort. | Skills are stateless reusable procedures; projects/memory carry evolving state (source: https://claude.com/resources/tutorials/teach-claude-your-way-of-working-using-skills). |
| **Plugin** | You want to distribute Skills + connectors + sub-agents together to a team/org. | The packaging/distribution unit above a Skill (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork; source: https://code.claude.com/docs/en/skills). |

Mental model (community, corroborates the official framing): *"Skills are workflows, MCP is data, subagents are plans, hooks are events. Pick by the layer."* (community: https://www.mindstudio.ai/blog/claude-code-skills-vs-slash-commands-2 / https://www.morphllm.com/claude-code-plugins-vs-skills).

Representative workflow examples Anthropic cites for Skills: weekly team updates (wins/blockers/priorities), customer-feedback analysis (categorization + pattern ID), sales-call prep (account research, talking points), brand compliance (bundled logos/colors/fonts/templates) (source: https://claude.com/resources/tutorials/teach-claude-your-way-of-working-using-skills).

---

## 9. A complete worked example (spec-verified)

```markdown
---
name: customer-feedback-synthesis
description: Categorize and synthesize customer feedback into themed insight reports with severity tags and recommended actions. Use whenever the user shares customer feedback, survey exports, support-ticket dumps, NPS verbatims, or asks for a feedback summary, voice-of-customer report, or theme analysis — even if they don't say "synthesis".
license: Proprietary. LICENSE.txt has complete terms
metadata:
  author: platform-team
  version: "1.2"
---

# Customer Feedback Synthesis

## Workflow
Copy this checklist and check items off as you go:
- [ ] 1. Load all feedback sources from the provided files/folder
- [ ] 2. Normalize and de-duplicate entries (run scripts/normalize.py)
- [ ] 3. Cluster into themes; tag each with severity (P0–P3)
- [ ] 4. Validate the theme map (run scripts/validate_themes.py)
- [ ] 5. Generate the report using the template
- [ ] 6. Verify every theme cites ≥1 source quote

## Step 2 — Normalize
Run: `python scripts/normalize.py <input> > normalized.json`
Do not hand-clean the data; the script handles encoding and dedupe deterministically.

## Step 5 — Report template
See [references/report-template.md](references/report-template.md) for the exact structure.

## Edge cases
- Empty/blank feedback rows: drop silently (handled by normalize.py).
- Mixed languages: keep original quote, add an English gloss in brackets.
```

This respects every official rule: gerund-ish focused name matching its folder, third-person "pushy" description with trigger keywords, body < 500 lines, one-level-deep reference, deterministic script with explicit instruction to *run* it, checklist workflow, validate→verify loop, no time-sensitive content, version in `metadata` (sources: §1.2, §2, §3 above).

---

## Sources

1. https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills — Anthropic engineering blog, "Equipping agents for the real world with Agent Skills." Backs: definition, progressive disclosure, code-vs-prose, security, evaluation-first iteration, open-standard. Published 2025-10-16; updated 2025-12-18 (open standard).
2. https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview — Official Agent Skills overview. Backs: SKILL.md structure, name/description constraints + reserved words, 3-level loading table, cross-surface non-sync, sharing scopes, runtime constraints (network/packages by surface), API beta headers, security, available pre-built skills. Living doc (no single date; references features through Dec 2025).
3. https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices — Official skill-authoring best practices. Backs: conciseness, degrees of freedom, naming (gerund), third-person descriptions, progressive-disclosure patterns, one-level references, workflows/feedback loops, solve-don't-punt, no voodoo constants, evals-first + JSON schema, Claude A/B method, anti-patterns, MCP tool naming, checklist. Living doc.
4. https://code.claude.com/docs/en/skills — Claude Code "Extend Claude with skills." Backs: filesystem locations + precedence, full extended frontmatter table, slash-command merge, `disable-model-invocation`/`user-invocable`, `allowed-tools` semantics, skill content lifecycle + compaction, listing budget, sharing scopes, plugins. Living doc.
5. https://agentskills.io/specification — Agent Skills open-standard specification. Backs: the authoritative frontmatter table (`name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`), exact constraints, name-must-match-directory, directory layout, progressive-disclosure token figures, validation tool. Living spec.
6. https://agentskills.io/home — Agent Skills standard overview + client showcase. Backs: open-standard status, cross-product adoption list, 3-stage discovery/activation/execution. Living.
7. https://github.com/anthropics/skills/blob/main/README.md — Anthropic public skills repo README. Backs: repo structure (skills/spec/template), bundled doc skills, two-field frontmatter, plugin marketplace install commands, partner (Notion) skills, demonstration disclaimer.
8. https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md — Anthropic's official skill-creator Skill. Backs: under-triggering + "pushy" descriptions, "when to use" goes in description, when Claude consults skills (complex vs trivial), Principle of Lack of Surprise, full eval/benchmark/description-optimization loop, Cowork-specific branches, packaging, explain-the-why.
9. https://claude.com/blog/skills — Anthropic Skills launch announcement. Backs: 2025-10-16 launch, composable/portable/efficient/powerful, where skills work, skill-creator, plugin marketplace, Agent SDK support.
10. https://claude.com/blog/organization-skills-and-directory — Anthropic announcement (2025-12-18). Backs: org-wide management, simplified creation, skills directory, partner skills (Atlassian/Canva/Figma/Cloudflare/Vercel/Sentry/Zapier), open standard.
11. https://support.claude.com/en/articles/12512176-what-are-skills — Help Center "What are Skills?" Backs: definition, plan availability, skill categories (Anthropic/Custom/Org-provisioned/Partner), `/skill-creator` in Cowork, Customize→Skills. Updated 2026-03-31.
12. https://support.claude.com/en/articles/12512180-use-skills-in-claude — Help Center "Use Skills in Claude." Backs: enable via Customize→Skills / Settings→Capabilities, zip upload, per-account privacy, org-provisioned (view-only, auto-update), Cowork directory access. Updated 2026-04-13.
13. https://support.claude.com/en/articles/12512198-how-to-create-custom-skills — Help Center "How to create custom Skills." Backs: step-by-step, ZIP root structure (`my-skill/SKILL.md`), scripts/languages, test-then-enable. Updated 2026-03-16. (Note: its "200-char description" / `dependencies` field conflict with the spec — see Confidence & gaps.)
14. https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — Help Center "Get started with Claude Cowork." Backs: Cowork architecture (Claude Code engine, isolated VM), plan→subtasks→parallel, folder instructions, `/schedule`, plugins bundle skills+connectors+sub-agents, controlled file/network egress. "Updated this week" at fetch (≈ May 2026).
15. https://support.claude.com/en/articles/13119606-provision-and-manage-skills-for-your-organization — Help Center org provisioning. Backs: Owner-only, Team/Enterprise, prerequisite toggles, exact provisioning steps, default-enabled, sharing controls (colleague + org directory, both default off, no approval workflow), audit log/Compliance API `role_assignment`. "Updated this week."
16. https://claude.com/resources/tutorials/teach-claude-your-way-of-working-using-skills — Anthropic tutorial. Backs: non-developer creation flow, Skills vs Projects vs Custom Instructions vs Prompting table, representative workflow examples. No explicit date.
17. https://sherlock.xyz/post/how-to-write-skills-for-claude-code-and-cowork — *Community* practitioner guide (Sherlock, 2026-03-10). Backs (corroborating, not primary): Cowork uses the same `~/.claude/skills/` and `.claude/skills/` filesystem locations as Claude Code; "pushy" descriptions; < 500 lines; `disable-model-invocation` for side-effecting skills.
18. https://www.mindstudio.ai/blog/claude-code-skills-vs-slash-commands-2 / https://www.morphllm.com/claude-code-plugins-vs-skills — *Community* — used only for the "skills=workflows, MCP=data, subagents=plans" mental-model framing in §8.

## Confidence & gaps

- **High confidence (official, multiple corroborating Anthropic sources):** SKILL.md structure; `name`/`description` constraints; the three-level progressive-disclosure model and token figures; all authoring best practices, anti-patterns, evals-first method, and Claude A/B iteration; cross-surface non-sync and per-surface sharing scopes; org provisioning rules and audit behavior; the skill-creator workflow; the open-standard frontmatter schema.
- **`version` field:** Confirmed there is **no top-level `version`** in the open standard — versioning lives in the free-form `metadata` map. One Help Center article ([13]) references a `dependencies` field and a "200-character" description max; these **conflict with the canonical spec** ([5], which says `description` ≤ 1024 chars and uses `compatibility` for environment needs). I treated the agentskills.io specification and the platform docs as authoritative; the Help Center figure is likely a simplified/older UI guideline. Flagged as a discrepancy rather than resolved.
- **`allowed-tools`:** Officially **experimental** in the open standard with implementation-varying support; fully specified only for Claude Code. Its exact behavior in claude.ai/Cowork is **not documented** by Anthropic — do not assume Cowork honors it the way Claude Code does.
- **Cowork skill *storage location* (`~/.claude/skills/`, `.claude/skills/`):** Anthropic states Cowork uses "the same agentic architecture that powers Claude Code" and supports Custom Skills, and the Cowork help article describes filesystem/folder behavior — but the **exact `.claude/skills/` paths for Cowork are documented authoritatively only for Claude Code** ([4]); Cowork parity is asserted by Anthropic at the architecture level and stated explicitly only by a community source ([17]). Treat the paths as highly likely but Cowork-undocumented at the path level.
- **Cowork "workflows":** Anthropic does **not** define a separate "workflow" object distinct from Skills; "skills" *are* the reusable workflows, and `/schedule` + folder instructions are the adjacent Cowork mechanisms. If you have seen a "Workflows" UI surface, that is not described in current public Anthropic documentation as of this research (potentially newer/internal — gap).
- **Dates:** Several Help Center articles report "Updated this week" rather than an absolute date at fetch time (≈ 2026-05-18); the engineering blog and launch posts have firm dates (2025-10-16, updated 2025-12-18). Living docs (overview, best-practices, Claude Code skills) carry no single date and should be re-checked for drift before high-stakes use.
- **Spec source — resolved 2026-05-18 via the Claude Chrome extension:** `spec/agent-skills-spec.md` in `anthropics/skills` is now a **one-line stub whose entire content is "The spec is now located at https://agentskills.io/specification."** So the canonical specification this doc relied on (agentskills.io/specification) *is* the authoritative source by Anthropic's own redirect — the earlier "raw file didn't load" gap is favorably closed, not an unverified assumption.

# Slash Commands & Customization Parity: Claude Code vs. Claude Cowork

*Why a Claude Code power user sees built-in slash commands that aren't in Cowork — and what to build instead.*

## TL;DR (the one-paragraph answer)

The confusion is real but explainable. **Most of the slash commands a Claude Code power user relies on are CLI/IDE session-control built-ins** (`/init`, `/clear`, `/compact`, `/model`, `/permissions`, `/hooks`, `/resume`, `/review`, `/security-review`, `/diff`, `/vim`, `/terminal-setup`, etc.). Their behavior is *coded into the Claude Code CLI* and is meaningless outside a terminal session, so they **do not exist in Cowork** (source: https://code.claude.com/docs/en/commands). What *does* port is the **customization layer** — *Skills* (and plugins that bundle Skills + connectors + sub-agents). Cowork **does have a `/` menu**, but it lists Skills/slash commands you install or write, not Claude Code's CLI built-ins (source: https://support.claude.com/en/articles/12512180-use-skills-in-claude). So the right mental model for a power user: stop hunting for built-in slash commands; build **Skills + Memory + connectors/plugins**, which are the portable, shared unit across Claude.ai, Cowork, Claude Code, and the API (source: https://support.claude.com/en/articles/12512176-what-are-skills).

---

## 1. Claude Code's slash command system

In Claude Code, typing `/` at the start of a message opens a command menu. Commands fall into **three categories** (source: https://code.claude.com/docs/en/commands, ref. v2.1.9x docs):

1. **Built-in commands** — behavior hard-coded into the CLI (model switching, context management, permissions, etc.).
2. **Bundled skills** — prompt-based capabilities Anthropic ships with the CLI, marked **Skill** in the reference; invoked like any skill (`/simplify`, `/batch`, `/debug`, `/loop`, `/claude-api`, `/fewer-permission-prompts`).
3. **User/project/plugin skills (formerly "custom slash commands")** — files you author.

### 1a. Catalogue of documented built-in commands

The official commands reference lists the full set. Highlights, grouped by purpose (source: https://code.claude.com/docs/en/commands):

| Command | What it does |
|---|---|
| `/init` | Generate a starter `CLAUDE.md` for a repo |
| `/memory` | Edit `CLAUDE.md` memory files; manage auto-memory |
| `/clear` (`/reset`, `/new`) | Start a new conversation with empty context |
| `/compact` | Summarize conversation to free context |
| `/context` | Visualize context-window usage |
| `/config` (`/settings`) | Open the Settings interface (theme, model, output style) |
| `/model` | Select/change model |
| `/effort` | Set model effort level |
| `/permissions` (`/allowed-tools`) | Manage allow/ask/deny tool permission rules |
| `/hooks` | View hook configurations for tool events |
| `/agents` | Manage subagent configurations |
| `/mcp` | Manage MCP server connections & OAuth |
| `/plugin` | Manage Claude Code plugins |
| `/skills` | List available skills; toggle visibility |
| `/review` | Review a pull request locally |
| `/security-review` | Scan branch git diff for security vulnerabilities |
| `/diff` | Interactive diff viewer for uncommitted changes |
| `/resume` (`/continue`) | Resume a conversation by ID/name |
| `/branch` (`/fork`) | Branch the current conversation |
| `/rewind` (`/checkpoint`, `/undo`) | Roll code/conversation back to a checkpoint |
| `/doctor` | Diagnose the Claude Code install |
| `/help` | Show help and available commands |
| `/terminal-setup` | Configure terminal keybindings (Shift+Enter, etc.) |
| `/vim` | *Removed in v2.1.92* — Vim mode now lives in `/config → Editor mode` |
| `/pr-comments` | *Removed in v2.1.91* — ask Claude directly instead |
| `/login` / `/logout` | Sign in/out of Anthropic account |
| `/cost` / `/usage` / `/stats` | Session cost, plan limits, activity |
| `/export` | Export conversation as plain text |
| `/schedule` (`/routines`) | Create/manage cloud-run routines (also exists in Cowork — see §2) |

Note two things a power user may not realize: `/vim` and `/pr-comments` were **recently removed/relocated** (source: https://code.claude.com/docs/en/commands), and **not every command appears for every user** — availability depends on platform, plan, and environment (e.g. `/desktop` is macOS/Windows only). This alone can explain some "they don't have it" cases *even between two Claude Code installs*.

### 1b. Where custom commands live, and the commands→skills merge

This is the single most important fact for understanding parity. **Custom slash commands have been merged into Skills** (source: https://code.claude.com/docs/en/skills, updated docs):

> "A file at `.claude/commands/deploy.md` and a skill at `.claude/skills/deploy/SKILL.md` both create `/deploy` and work the same way."

Storage locations (source: https://code.claude.com/docs/en/skills):

| Location | Path | Applies to |
|---|---|---|
| Enterprise | via managed settings | All users in the org |
| Personal | `~/.claude/skills/<name>/SKILL.md` | All your projects |
| Project | `.claude/skills/<name>/SKILL.md` | This project only |
| Plugin | `<plugin>/skills/<name>/SKILL.md` | Where the plugin is enabled |
| *Legacy* | `.claude/commands/*.md` (project) / `~/.claude/commands/*.md` (personal) | Still works; skills take precedence on name clash |

**Frontmatter** (YAML between `---` markers in `SKILL.md`): `name`, `description`, `when_to_use`, `argument-hint`, `arguments`, `disable-model-invocation`, `user-invocable`, `allowed-tools`, `model`, `effort`, `context`, `agent`, `hooks`, `paths`, `shell` (source: https://code.claude.com/docs/en/skills).

**Arguments:** `$ARGUMENTS` (full string), `$ARGUMENTS[N]` / `$N` (positional), `$name` (named via `arguments:` frontmatter); plus dynamic context injection via `` !`command` `` and file refs (source: https://code.claude.com/docs/en/skills).

**Plugins add commands** by namespacing: a `hello/` skill in plugin `my-first-plugin` becomes `/my-first-plugin:hello`. Plugins are installed via `/plugin` or a marketplace and can also bundle agents, hooks, MCP servers, LSP servers, and monitors (source: https://code.claude.com/docs/en/plugins). **MCP servers** can also expose prompts that appear as `/mcp__<server>__<prompt>` commands (source: https://code.claude.com/docs/en/commands).

---

## 2. Claude Cowork's command/skill surface

**What Cowork is:** Cowork brings "Claude Code's agentic capabilities to Claude Desktop for knowledge work beyond coding." It runs **only in the Claude Desktop app (macOS/Windows)** — not web, not mobile — and is for paid plans (Pro, Max, Team, Enterprise) (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork, updated ~May 2026).

**Does Cowork expose slash commands? Yes — but a different set.** This is the precise resolution of the confusion:

- Cowork **has a `/` menu**: "type `/` in the sidebar to see available Skills and select one (for example, `/debug` or `/deck-check`)" and you can "type `/` in the prompt box or click the `+` button and select **Slash commands** to browse what's available" (source: https://support.claude.com/en/articles/12512180-use-skills-in-claude, pub. 2026-04-13).
- But that menu surfaces **Skills and plugin slash commands** — the *customization* layer — **not Claude Code's CLI built-ins**. There is no `/init`, `/clear`, `/compact`, `/model`, `/permissions`, `/hooks`, `/diff`, `/resume`, `/terminal-setup` in Cowork; those are CLI session-control commands with no terminal to control (source: https://claude.com/resources/tutorials/customize-claude-cowork; absence corroborated by https://code.claude.com/docs/en/commands which defines them as CLI built-ins).
- The **primary** Cowork interaction model is **natural language**: "Claude applies relevant Skills automatically while you work — you don't need to invoke them separately… or describe your task naturally" (source: https://support.claude.com/en/articles/12512180-use-skills-in-claude).
- One genuinely **shared** command: `/schedule` exists in *both* Cowork (type `/schedule` or click "Scheduled" in the sidebar) and Claude Code (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork; https://code.claude.com/docs/en/commands).
- Cowork management happens through a **Customize** sidebar (Skills, Connectors, Plugins, Instructions), with a unified directory to **Browse**/**Install** skills, connectors, and plugins via the `+` button (source: https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork, updated 2026-04-09; https://support.claude.com/en/articles/14328846-browse-skills-connectors-and-plugins-in-one-directory, 2026-03-31).

> Practical note: plugin slash commands like `/sales:call-prep` are documented as "available in your session" for **Claude Code**; Anthropic's `knowledge-work-plugins` repo is "Built for Claude Cowork, also compatible with Claude Code," and in Cowork the help center's consistent framing is that **skills fire automatically or you select them from the `/` menu / `+` → Slash commands**, rather than typed CLI invocation being the headline path (source: https://github.com/anthropics/knowledge-work-plugins; https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork).

---

## 3. Mapping table: Claude Code construct → Cowork equivalent

| Claude Code construct | Cowork equivalent | Portable? | Source |
|---|---|---|---|
| CLI session built-ins: `/clear`, `/compact`, `/context`, `/resume`, `/branch`, `/rewind` | **None** (no CLI session to manage; Cowork manages context itself) | No | https://code.claude.com/docs/en/commands |
| Repo/dev built-ins: `/init`, `/review`, `/security-review`, `/diff`, `/pr-comments` | **None** (no local repo / git workflow surface in Cowork) | No | https://code.claude.com/docs/en/commands |
| Config built-ins: `/config`, `/model`, `/effort`, `/permissions`, `/hooks`, `/terminal-setup`, `/vim` | Replaced by **Customize sidebar** UI + **Instructions**; no per-session model/permission/hook commands | No (different mechanism) | https://claude.com/resources/tutorials/customize-claude-cowork |
| `/schedule` (routines) | **`/schedule`** / "Scheduled" sidebar item | **Yes** (same concept, both surfaces) | https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork |
| Custom slash commands (`.claude/commands/*.md`, `~/.claude/commands`) | **Skills** (`SKILL.md`) installed/created in Cowork; appear in `/` menu | **Yes** — commands merged into Skills; same `SKILL.md` standard | https://code.claude.com/docs/en/skills |
| Bundled skills (`/simplify`, `/debug`, `/batch`, `/loop`) | Skills surface; `/debug`, `/deck-check` shown as Cowork `/` examples | Partly — skill *mechanism* ports; the specific dev-oriented ones may not be useful in Cowork | https://support.claude.com/en/articles/12512180-use-skills-in-claude |
| `/mcp` (manage MCP servers) | **Connectors** (managed via Customize sidebar) | Concept ports; UI not a command | https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork |
| `/agents` (subagents) | Sub-agents bundled inside **plugins** | Concept ports; no `/agents` manager command | https://claude.com/blog/cowork-plugins |
| `/plugin` (plugin manager) | **Customize → Plugins → Browse/Install** | Concept ports; UI not a typed command | https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork |
| `/memory` + `CLAUDE.md` | **Memory** (project-scoped) + **Instructions** | Partly — Memory is "supported within projects but not retained across standalone Cowork sessions" | https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork |
| Hooks (`settings.json` / `hooks.json`) | **No equivalent** | **No** | https://code.claude.com/docs/en/plugins |
| `settings.json` / CLI flags / `--plugin-dir` | **No equivalent** (managed product) | **No** | https://code.claude.com/docs/en/plugins |

---

## 4. Why the divergence exists (grounded in sources)

The split is *by design*, not an oversight:

- **Claude Code is a developer CLI/IDE tool** with a config-file + plugin ecosystem. Its commands manage a terminal session and a code repository: context windows (`/compact`, `/clear`), models/effort (`/model`, `/effort`), permissions and hooks (`/permissions`, `/hooks`), git/PR work (`/review`, `/security-review`, `/diff`), and session lifecycle (`/resume`, `/branch`, `/rewind`). These are "built-in command[s] whose behavior is coded into the CLI" (source: https://code.claude.com/docs/en/commands). A managed desktop product has no terminal session to clear, no local git branch to diff, no `settings.json` for the user to edit — so these commands **have no referent** in Cowork.

- **Cowork is a managed/agentic product surface for non-CLI users.** It uses "the same agentic architecture that powers Claude Code" but exposes it through Claude Desktop with a natural-language-first model and a **Customize** UI instead of config files (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork; https://claude.com/blog/cowork-plugins, pub. 2026-01-30).

- **The shared concepts deliberately appear differently.** Skills, connectors (MCP), sub-agents, memory, and projects exist in both — but in Cowork they're surfaced via UI panels and automatic/`/`-menu invocation rather than CLI commands and dotfiles (sources above).

So a power user's muscle memory for `/init` or `/compact` *should* fail in Cowork; those aren't features Cowork is missing — they're CLI plumbing Cowork doesn't need.

---

## 5. Skills/plugins: the shared customization layer

This is what *does* port, and it's the answer to "what should they build."

**Skills are the portable unit.** Skills are available in **Claude.ai (all plans), Cowork, Claude Code (beta), and all API users with code execution enabled** (source: https://support.claude.com/en/articles/12512180-use-skills-in-claude, 2026-04-13). They follow the **Agent Skills open standard published at agentskills.io**: "skills you create aren't locked to Claude — the same skill format works across AI platforms and tools that adopt the standard," with a reference Python SDK (source: https://support.claude.com/en/articles/12512176-what-are-skills, 2026-03-31; https://code.claude.com/docs/en/skills).

A `SKILL.md` written for Claude Code is the **same artifact** Cowork installs. Plugins bundle Skills + connectors + slash commands + sub-agents into one installable package, distributed via the unified directory / marketplace (source: https://github.com/anthropics/knowledge-work-plugins; https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork).

**What does NOT port:**
- **Hooks** — Claude Code-only event handlers (`settings.json` / `hooks.json`); no Cowork equivalent (source: https://code.claude.com/docs/en/plugins).
- **`settings.json` and CLI flags** (`--plugin-dir`, `--add-dir`, env vars like `CLAUDE_CODE_*`) — Claude Code-only (source: https://code.claude.com/docs/en/plugins; https://code.claude.com/docs/en/skills).
- **CLI-only built-in commands** — `/init`, `/clear`, `/compact`, `/diff`, `/resume`, `/terminal-setup`, `/vim`, etc. (source: https://code.claude.com/docs/en/commands).
- **Memory durability differs** — Cowork memory is project-scoped and "not retained across standalone Cowork sessions," unlike Claude Code's persistent `CLAUDE.md` (source: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).

---

## 6. Practical guidance

The right mental model: **don't look for built-in slash commands in Cowork. Build the portable layer instead.**

1. **Stop equating "slash command" with "built-in."** In Cowork, the `/` menu and `+` → "Slash commands" surface **Skills and installed plugin commands**, not CLI plumbing. Type `/` to see what *they* actually have (source: https://support.claude.com/en/articles/12512180-use-skills-in-claude).
2. **Build Skills, not commands.** Anything previously encoded as a custom `.claude/commands/*.md` is now just a `SKILL.md` — author it once, it works in Cowork *and* Claude Code *and* Claude.ai (source: https://code.claude.com/docs/en/skills). You can even create a skill conversationally (source: https://support.claude.com/en/articles/12599426-how-to-create-a-skill-with-claude-through-conversation).
3. **Use Instructions + Memory for standing context** instead of `CLAUDE.md`/`/memory` — set in the Customize sidebar; remember Cowork memory is project-scoped (source: https://claude.com/resources/tutorials/customize-claude-cowork; https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork).
4. **Use Connectors instead of `/mcp`** to wire in Slack, Google Drive, Salesforce, Jira, etc. via the Customize sidebar (source: https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork).
5. **Install role plugins from the directory.** Anthropic's `knowledge-work-plugins` ("Built for Claude Cowork") ship ready-made skills/connectors/sub-agents for Sales, Finance, Product, Data, etc. — install via Customize → Plugins → Browse (source: https://github.com/anthropics/knowledge-work-plugins; https://claude.com/blog/cowork-plugins).
6. **Lead with natural language.** Cowork's intended path is "describe the task" — skills auto-fire. The `/` picker is a convenience, not the primary interface (source: https://support.claude.com/en/articles/12512180-use-skills-in-claude).
7. **Set the expectation:** the overlap is *Skills + connectors + sub-agents + `/schedule` + projects/memory*. CLI session/repo/config commands and hooks are Claude Code-only and that's intentional.

---

## Sources

1. https://code.claude.com/docs/en/commands — Full Claude Code built-in command & bundled-skill catalogue; built-in vs Skill distinction; per-platform availability; `/vim` & `/pr-comments` removals. Reflects v2.1.9x-era docs (fetched 2026-05-18).
2. https://code.claude.com/docs/en/skills — Skills/SKILL.md format, frontmatter, arguments, storage locations, commands→skills merge, legacy `.claude/commands`, Agent Skills open standard. (fetched 2026-05-18; redirected from the old `docs.claude.com/en/docs/claude-code/slash-commands`).
3. https://code.claude.com/docs/en/plugins — Plugin structure, manifest, namespacing, marketplace install, what plugins bundle; hooks/`settings.json` as Claude Code-only (fetched 2026-05-18).
4. https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork — What Cowork is, desktop-only, plans, `/schedule`, plugins/connectors, project-scoped memory, relation to Claude Code. Updated ~May 2026 ("updated this week").
5. https://support.claude.com/en/articles/12512180-use-skills-in-claude — Skills across Claude.ai/Cowork/Claude Code/API; `/` menu & `+` → Slash commands in Cowork; automatic vs manual invocation; Customize > Skills. Pub. 2026-04-13.
6. https://support.claude.com/en/articles/12512176-what-are-skills — Skills definition; supported products; portability via agentskills.io open standard; skill types. Pub. 2026-03-31.
7. https://support.claude.com/en/articles/13837440-use-plugins-in-claude-cowork — Cowork plugins bundle skills/connectors/sub-agents; install via Customize sidebar; paid plans. Updated 2026-04-09.
8. https://support.claude.com/en/articles/14328846-browse-skills-connectors-and-plugins-in-one-directory — Unified directory; Customize sidebar browse/install; view-only directory skills. Pub. 2026-03-31.
9. https://github.com/anthropics/knowledge-work-plugins — Anthropic-official plugin repo "Built for Claude Cowork, also compatible with Claude Code"; plugin components incl. slash commands; install from claude.com/plugins (fetched 2026-05-18).
10. https://claude.com/blog/cowork-plugins — Anthropic blog: Cowork customized via plugins bundling skills/connectors/slash commands/sub-agents; file-based; natural-language goal-setting. Pub. 2026-01-30.
11. https://claude.com/resources/tutorials/customize-claude-cowork — Cowork skill invocation (`/skill-name` or natural language), Customize sidebar (Connectors/Instructions/Skills/Plugins); no CLI built-ins mentioned (fetched 2026-05-18; no visible date).

*All sources above are **Anthropic-official** (docs.claude.com / code.claude.com, support.claude.com Help Center, claude.com blog/tutorials, and the official `anthropics/` GitHub org). No third-party/community sources were used in the final claims.*

---

## Confidence & gaps

- **High confidence:** Claude Code's built-in command catalogue and the commands→Skills merge (directly quoted from official docs, source 1 & 2). The structural reason for divergence (CLI vs managed product) — explicit in sources 1, 4, 10.
- **High confidence:** Skills are the portable unit across Claude.ai/Cowork/Claude Code/API via the agentskills.io open standard (sources 5, 6).
- **Medium-high confidence:** Cowork *does* have a `/` menu listing Skills/slash commands (source 5 explicitly: "type `/` in the prompt box… select Slash commands to browse"). One nuance: source 5's clearest "type `/`" wording is given in the context of the Excel/PowerPoint sidebar add-ins; the Cowork get-started/customize articles (4, 11) emphasize natural-language + `/skill-name` + `+`/Customize sidebar and do not enumerate a broad `/`-built-in menu. Net: Cowork has slash *skill/command* invocation, but Anthropic documents the **natural-language path as primary** and does **not** document Claude Code CLI built-ins in Cowork.
- **Version drift caveat:** Claude Code command lists change fast (`/vim`, `/pr-comments` removed in v2.1.91–92 per source 1). Treat the §1a catalogue as accurate to the fetched docs (2026-05-18), not permanently fixed. "Not every command appears for every user" (plan/platform-gated) is itself a documented reason two users see different commands.
- **Gap:** No single official page exhaustively states "these exact Claude Code built-ins are absent in Cowork." That conclusion is **derived** (sound, but inferential) from (a) source 1 defining them as CLI built-ins and (b) sources 4/10/11 describing Cowork's UI-driven, no-CLI model. Flagged rather than asserted as a direct quote.
- **Gap:** Exact Cowork plugin-slash-command trigger UX (typed `/sales:call-prep` vs picker) is documented crisply for Claude Code (source 9) but only generally for Cowork; precise Cowork keystroke behavior for namespaced plugin commands was not found verbatim.
- **Minor:** Some help-center articles expose only relative dates ("updated this week"); absolute dates are given where the page showed them.

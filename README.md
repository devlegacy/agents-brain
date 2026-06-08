# agents-brain

A Claude Code skill that installs an **LLM Wiki** (an Obsidian-style knowledge base) in any project. A single command and Claude configures everything: an interconnected markdown wiki with session nodes, a dense index, an append-only log, and a slash command (`/brain`) for ingest, query, and lint operations.

## What you get

After running the skill, your project will have:

```
brain/                        ← wiki directory (the name is configurable)
├── CLAUDE.md                 ← operational schema (rules for Claude)
├── INDEX.md                  ← dense node catalog (LLM cursor)
├── LOG.md                    ← append-only operations log
├── SOURCES.md                ← external sources registry
├── sessions/                 ← session nodes, one per conversation
└── plannings/                ← planning nodes, living documents across sessions
.claude/commands/brain.md     ← /brain slash command
.vscode/settings.json         ← wikilink configuration for Foam
```

The wiki is maintained by Claude through four commands:

| Command | What it does |
|---------|--------------|
| `/brain ingest` | Creates a session node from the current conversation. Extracts decisions, outputs, cross-references, and pending items. Updates the index, log, and any related planning nodes automatically. |
| `/brain query <question>` | Answers using the wiki as a source, with `[[wikilink]]` citations. Fetches updated content from external sources when necessary. |
| `/brain lint` | Health check: broken wikilinks, orphan nodes, outdated claims, missing cross-references, stale plannings, candidates for new concepts. Read-only. |
| `/brain plan <slug>` | Creates or updates a planning node — a living document that tracks tasks, progress, and the sessions that created and advanced the plan. |

## Prerequisites

Before installing, make sure you have:

- **[Claude Code](https://claude.ai/code)** — Anthropic's CLI. This is the environment where skills and slash commands run. Without this, nothing works.
- **[Foam for VSCode](https://marketplace.visualstudio.com/items?itemName=foam.foam-vscode)** — optional but highly recommended if you want to visualize the graph of your wiki (see below).
- **Notion MCP** — only if you plan to use Notion as your source of truth (see the configuration section).
- **Git** — only if you want to clone the skill's repository.

## How to install

### Step 1 — Copy the skill to your project

You have two options depending on whether you want to use it in one project or all projects:

```bash
# For a specific project (run this inside your project directory)
git clone https://github.com/devlegacy/agents-brain-skill .agents/skills/agents-brain-skill

# For global use (available in all your projects)
git clone https://github.com/devlegacy/agents-brain-skill ~/.claude/skills/agents-brain-skill
```

### Step 2 — Run the skill from Claude Code

Open Claude Code in your project and run:

```
/agents-brain
```

Claude will ask you the configuration questions (it can ask them all at once) and will automatically generate the entire structure.

> If the `/agents-brain` command doesn't appear, verify that the skill is in the correct folder: `.agents/skills/agents-brain-skill/` (for the project) or `~/.claude/skills/agents-brain-skill/` (global).

## Configuration questions

When running `/agents-brain`, Claude will ask you the following:

### 1. Wiki directory name
What to call the main folder. The default is `brain`, but you can use `wiki`, `knowledge`, `memory`, etc.

### 2. Slash command name
The command you will use to operate the wiki. The default is the same as the directory name. For example, if you choose `memory`, the commands would be `/memory ingest`, `/memory query`, `/memory lint`, `/memory plan`.

### 3. Project name
Used in the internal description of the command. It can be the name of your company, team, or product.

### 4. Content language *(optional)*
**English is the default** — leave blank to use it. Only specify a language if you need something else (Spanish or other). This affects the internal section headers of each node (Context, Decisions, Pending, etc.).

### 5. Project areas
The work categories for your team. For example: `marketing, ops, dev, research, product`. These are used as sections in the index and as values of the `area` field in each node's frontmatter.

### 6. Source of truth backend
Where the project knowledge comes from:

- **Notion** — the wiki stores pointers to Notion pages with short IDs. Claude fetches the updated content via MCP when needed. Requires the Notion MCP configured in `.mcp.json`.
- **Local files** — the source of truth consists of files inside the same repository. Claude reads them directly with Read.
- **Another tool** — Confluence, Linear, Airtable, etc. You describe how it works and Claude adapts the instructions.
- **None** — standalone wiki, without an external source. Claude answers only from what is in the nodes.

### 7. Historical sessions
If you have logs, notes, or changelogs from previous sessions to migrate, Claude generates an immutability note in the schema. You can then migrate those files manually by adding the corresponding YAML frontmatter.

## How it works

Each session node is a markdown file with YAML frontmatter and six fixed sections:

```yaml
---
type: session
area: marketing
date: 2026-04-10
slug: email-infrastructure-setup
title: "Email Capture Infrastructure: Brevo + n8n + Nginx"
tags: [email, lead-magnet, brevo, n8n, nginx]
status: active
related:
  - 2026-04-04-cold-email-sequence
sources:
  - notion:email-strategy
  - repo:.claude/plans/email-infra.md
superseded_by: null
---

## Context
## Decisions
## Output
## Pending
## Cross-refs
- [[2026-04-04-cold-email-sequence]] — same date, complementary piece
## Sources
- [[sources#email-strategy]] (Notion)
```

Internal links use the `[[slug]]` syntax (wikilinks). The index (`INDEX.md`) is the main cursor: Claude reads it first in each operation to decide which nodes to open, without reading the entire wiki at once.

## Graph visualization with Foam

To see the visual graph of your wiki, install the **Foam** extension in Visual Studio Code:

1. Open the extensions panel (`Ctrl+Shift+X` on Windows/Linux, `Cmd+Shift+X` on Mac)
2. Search for **"Foam"** and select the extension from the "Foam team"
3. Install it and reload VSCode
4. With your project open, use the **"Foam: Show Graph"** command from the command palette (`Ctrl+Shift+P` / `Cmd+Shift+P`)

With Foam you will have:
- Clickable `[[wikilinks]]` in the editor
- Interactive visual graph of all nodes and their connections
- Backlinks panel (which nodes point to the current node)
- Autocomplete when typing `[[`

The skill configures `.vscode/settings.json` automatically so that the graph does not include unnecessary folders like `.claude/` or legacy sessions.

## Notion configuration (only if using that backend)

If you chose Notion as your source of truth, you need to:

1. Configure the **Notion MCP** in the `.mcp.json` file of your project
2. Fill in `brain/SOURCES.md` with the short IDs and URLs of your Notion pages

Claude will automatically fetch Notion content when it needs it during a query.

## The pattern

This skill implements the **LLM Wiki - Karpathy pattern**: a persistent, self-compiled knowledge base where the LLM is both the writer and the reader. The key is that Claude reads the index first in each operation, making retrieval O(index) rather than O(all-files). Cross-references are bidirectional and are verified in each ingest.

The three-layer architecture:
1. **Raw sources** — Notion pages, repo files (immutable, fetched fresh)
2. **Wiki** — `brain/` — session nodes with frontmatter, dense index, cross-references
3. **Schema** — `brain/CLAUDE.md` — the operational manual Claude reads before each operation

## License

MIT

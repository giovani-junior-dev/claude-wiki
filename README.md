# Claude Wiki

**Persistent knowledge base for Claude Code projects.**
Based on the [LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) methodology by Andrej Karpathy.

---

## The Problem

Every time you start a new Claude Code session, your AI assistant starts from zero. No memory of what was discussed yesterday. No knowledge of decisions made last week. No awareness of lessons learned last month.

**Claude Wiki fixes this.** One command. 30 seconds. Your project has a brain.

## What It Does

Claude Wiki bootstraps a structured wiki inside your project — interlinked Markdown files with YAML frontmatter, cross-references, and automatic mirroring to [Obsidian](https://obsidian.md/) via MCP.

Instead of rediscovering context from scratch each session, Claude builds and maintains a cumulative knowledge base:

- **Ingests sources** — articles, docs, notes dropped into `raw/`
- **Extracts entities** — technologies, libraries, APIs, services
- **Maps concepts** — patterns, architectures, methodologies
- **Records decisions** — context, options considered, trade-offs
- **Preserves learnings** — bug fixes, insights, root causes
- **Answers queries** — synthesized responses with citations to wiki pages

## Architecture

```
┌──────────┐    ┌──────────┐    ┌───────────┐
│  raw/    │    │  wiki/   │    │ CLAUDE.md │
│          │    │          │    │           │
│ Immutable│───>│ Generated│<──>│  Schema   │
│ Sources  │    │ by Claude│    │  (Rules)  │
│          │    │          │    │           │
└──────────┘    └──────────┘    └───────────┘
 You write       Claude          Co-evolved
 Claude reads    owns this       between you
 NEVER modifies  entirely        and Claude

                ┌──────────────────────┐
                │   Obsidian Vault     │
                │   (Mirror via MCP)   │
                └──────────────────────┘
```

**Three layers:**

1. **raw/** — Immutable source material. You drop files here. Claude reads but never modifies.
2. **wiki/** — Pages Claude generates and maintains. Summaries, entities, concepts, decisions, learnings.
3. **CLAUDE.md** — The schema that defines how the wiki operates. Co-evolved between you and Claude.

**Bonus:** Everything is dual-written to your Obsidian vault via MCP for graph view, search, and visual navigation.

## Install

```bash
claude install-skill giovani-junior-dev/claude-wiki
```

Then open any project and say:

```
"Set up a wiki for this project"
```

That's it. Claude will:

- Detect Obsidian MCP (offer to install if needed)
- Create `wiki/` and `raw/` directories with all subfolders
- Initialize `index.md` and `log.md`
- Configure `CLAUDE.md` with all wiki rules
- Mirror everything to Obsidian

### New Projects

```bash
cd my-new-project
# "Set up a wiki for this project"
# Done. 30 seconds.
```

### Existing Projects

```bash
cd my-existing-saas
# "Set up a wiki for this project"
# Creates wiki/ and raw/ alongside your existing code.
# Appends rules to CLAUDE.md — never overwrites.
# Your code, configs, and git history stay untouched.
```

**Pro tip:** After setup, drop your existing docs into `raw/docs/` and tell Claude: *"ingest everything in raw/"*. In minutes, you'll have a structured knowledge base built from your own documentation.

## Commands

| Command | Description |
|---------|-------------|
| `wiki ingest <file>` | Process a source from `raw/` into structured wiki pages |
| `wiki ingest all` | Ingest all new sources in `raw/` |
| `wiki query <question>` | Search the wiki and synthesize an answer with citations |
| `wiki status` | Show stats: pages by type, pending sources, last operations |
| `wiki lint` | Health check: contradictions, orphan pages, missing links |
| `wiki decision <title>` | Record an architectural decision with context and trade-offs |
| `wiki learning <title>` | Record a lesson learned from a bug fix or insight |
| `wiki search <term>` | Search across wiki pages |
| `wiki sync` | Force sync all local pages to Obsidian vault |

You don't need to memorize commands. Claude also responds to natural language:

> *"I just fixed a nasty caching bug, save it as a lesson"*

works just as well as `wiki learning caching bug`.

## Automatic Behaviors

Claude doesn't just wait for commands. With the wiki active, it also:

- Reads the wiki at the **start of every session**
- Asks to record **decisions** after architecture discussions
- Asks to record **learnings** after complex bug fixes
- **Alerts you** when new info contradicts existing wiki knowledge
- Asks to **ingest** new files dropped in `raw/`

## Wiki Page Format

Every wiki page has YAML frontmatter:

```yaml
---
title: "Page Name"
type: source | entity | concept | decision | learning | query
created: 2026-04-07
updated: 2026-04-07
tags: [architecture, hexagonal]
sources: [raw/articles/hexagonal-arch.md]
related: [clean-architecture, onion-architecture]
---
```

Pages use `[[wikilinks]]` for cross-references and kebab-case filenames.

## Obsidian Integration

Claude Wiki integrates with Obsidian via the [mcpvault](https://github.com/bitbonsai/mcpvault) MCP server. During bootstrap, if the MCP isn't detected, Claude offers to install it:

```bash
claude mcp add obsidian -- npx @bitbonsai/mcpvault@latest /path/to/your/vault
```

**Requirements:** Node.js v18+. No Obsidian plugins needed — connects directly to the vault folder.

**What you get:** Graph view of all wiki pages, full-text search, tag filtering, and visual navigation. Each project lives under its own folder: `YourProject/wiki/` and `YourProject/raw/`.

**Don't use Obsidian?** No problem. The skill works in local-only mode. You can add Obsidian later and run `wiki sync` to push everything.

## Tested

The skill was evaluated across 3 scenarios (bootstrap, ingest, sync) with automated grading:

| Scenario | With Skill | Without Skill |
|----------|-----------|---------------|
| Bootstrap | **100%** (7/7) | 29% (2/7) |
| Ingest | **100%** (8/8) | 50% (4/8) |
| Status | **100%** (5/5) | 40% (2/5) |
| **Average** | **100%** | **40%** |

Without the skill, Claude invents a generic wiki structure. With the skill, it follows the Karpathy methodology precisely — correct directory structure, YAML frontmatter, cross-references, Obsidian sync, operation logging.

## Origin

This skill was created after reading Andrej Karpathy's [LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) article, where the former Tesla AI Director proposed a pattern for building persistent knowledge bases with LLMs.

His core insight: instead of retrieving raw documents on every query, let the LLM **build and maintain a structured wiki** that compounds over time. The tedious bookkeeping — updating cross-references, keeping summaries current, noting contradictions — is exactly what LLMs excel at.

Claude Wiki turns that methodology into a one-command skill that any developer can use.

> *"The tedious bookkeeping — updating cross-references, keeping summaries current, noting contradictions — is what causes knowledge bases to fail. LLMs don't get bored."*
> — Andrej Karpathy

## Requirements

| Requirement | Status |
|-------------|--------|
| Claude Code | Required |
| Obsidian + MCP | Optional (enhances with graph view) |
| Node.js v18+ | Required for Obsidian MCP |

## License

MIT

## Author

**MadeInLowCode** — [GitHub](https://github.com/madeinlowcode)

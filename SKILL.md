---
name: claude-wiki
description: >
  Persistent knowledge base for Claude Code projects using the LLM Wiki methodology by Andrej Karpathy.
  Bootstraps a structured wiki (markdown files with cross-references) and configures Claude Code to maintain it
  automatically across sessions. Integrates with Obsidian via MCP for graph view and visual navigation.
  Use this skill whenever the user wants to: set up a project wiki, create persistent memory for a project,
  use the Karpathy wiki method, organize knowledge with Obsidian integration, bootstrap a knowledge base,
  or mentions "wiki", "knowledge base", "persistent memory", "LLM wiki", or "Karpathy wiki".
  Also trigger when the user says "wiki ingest", "wiki query", "wiki lint", "wiki status", "wiki decision",
  "wiki learning", "wiki search", or "wiki sync".
---

# Claude Wiki

Persistent, cumulative knowledge base for Claude Code projects.
Based on the **LLM Wiki** methodology by Andrej Karpathy — where an LLM builds and maintains
a structured wiki of interlinked markdown files instead of rediscovering knowledge from scratch each session.

## How It Works

There are three layers:

1. **raw/** — Immutable source material (articles, docs, notes). You read but NEVER modify.
2. **wiki/** — Pages you generate and maintain (summaries, entities, concepts, decisions, learnings). You own this layer entirely.
3. **CLAUDE.md** — The schema that defines how the wiki operates. Co-evolved between you and the user.

The wiki lives in two places simultaneously (dual-write):
- **Local**: `wiki/` and `raw/` folders in the project directory
- **Obsidian**: Same structure mirrored in the user's Obsidian vault via MCP, under `<ProjectName>/wiki/` and `<ProjectName>/raw/`

This gives the user Obsidian's graph view, search, and tagging while keeping local files as the source of truth that works offline.

---

## Bootstrap (First-Time Setup)

When the user activates this skill on a project for the first time, execute ALL of the following steps:

### Step 1: Check Obsidian MCP Availability

Before anything else, verify that the Obsidian MCP server is available. This is essential for the
dual-write functionality that mirrors the wiki to Obsidian.

**How to detect:** Try calling any Obsidian MCP tool (e.g., `mcp__obsidian__get_vault_stats`).

**If it works:** Great — the MCP is connected. Proceed to Step 2.

**If it fails or the tools are not available:**

Tell the user:

> The Obsidian MCP server is not configured yet. The wiki needs it to mirror pages to your Obsidian vault.
> I can set it up for you — it takes about 30 seconds. May I proceed?

**If the user says yes**, install it by running:

```bash
claude mcp add obsidian -- npx @bitbonsai/mcpvault@latest /path/to/your/vault
```

But first, ask the user for their Obsidian vault path. Example prompt:

> What is the path to your Obsidian vault? For example:
> - macOS: `~/Documents/MyVault`
> - Windows: `C:\Users\YourName\Documents\MyVault`
> - Linux: `~/obsidian-vault`

Once the user provides the path, run the command with the actual path:

```bash
claude mcp add obsidian -- npx @bitbonsai/mcpvault@latest <USER_VAULT_PATH>
```

Then inform the user:

> MCP server installed. For it to take effect, you need to:
> 1. **Restart Claude Code** (close and reopen the terminal session)
> 2. After restarting, run this skill again and the Obsidian integration will be active
>
> The MCP server is based on [mcpvault](https://github.com/bitbonsai/mcpvault) and requires Node.js v18+.
> It connects directly to your vault folder — no Obsidian plugins needed.

**If the user says no or wants to skip Obsidian:**

Proceed with the bootstrap but in **local-only mode**. Create all local files normally but skip
all Obsidian MCP calls. Add a note in the CLAUDE.md wiki rules section:

```
Note: Obsidian integration is not configured. The wiki operates in local-only mode.
To enable Obsidian sync later, ask Claude to set up the Obsidian MCP, or run:
claude mcp add obsidian -- npx @bitbonsai/mcpvault@latest /path/to/your/vault
Then restart Claude Code and run `wiki sync` to push all local pages to Obsidian.
```

### Step 2: Detect Project Name

Use the current directory name as the project name. Ask the user to confirm or provide a custom name.
This name will be used as the root folder in Obsidian: `<ProjectName>/wiki/`, `<ProjectName>/raw/`.

### Step 3: Create Local Directory Structure

Create these directories and files in the project root:

```
wiki/
├── index.md
├── log.md
├── sources/
├── entities/
├── concepts/
├── decisions/
├── learnings/
└── queries/

raw/
├── articles/
├── docs/
├── notes/
└── assets/
```

### Step 4: Initialize wiki/index.md

```markdown
---
title: "Wiki Index"
type: index
created: <TODAY>
updated: <TODAY>
tags: [wiki, index]
---

# Wiki Index

> Central catalog. Automatically updated on every ingest.
> Last updated: <TODAY>

## Sources
_No sources ingested yet._

## Entities
_No entities registered yet._

## Concepts
_No concepts registered yet._

## Decisions
_No decisions recorded yet._

## Learnings
_No learnings recorded yet._

## Queries
_No queries archived yet._
```

### Step 5: Initialize wiki/log.md

```markdown
---
title: "Wiki Log"
type: log
created: <TODAY>
updated: <TODAY>
tags: [wiki, log]
---

# Wiki Log

> Chronological record of all wiki operations (append-only).

## [<TODAY>] init | Wiki initialized
- Directory structure created
- CLAUDE.md configured with wiki rules
- Obsidian vault mirror created
- Ready for source ingestion
```

### Step 6: Create .gitignore for raw/assets

```
# Large binary files in raw/assets — keep out of git if needed
# raw/assets/*.png
# raw/assets/*.jpg
# raw/assets/*.pdf
```

Write this to `raw/.gitignore` only if one doesn't exist.

### Step 7: Mirror to Obsidian (skip if local-only mode)

Using the Obsidian MCP tools, create the same structure in the vault:

1. Use `mcp__obsidian__write_note` to create `<ProjectName>/wiki/index.md` with the same content
2. Use `mcp__obsidian__write_note` to create `<ProjectName>/wiki/log.md` with the same content
3. Create placeholder notes in each subfolder so Obsidian recognizes the folders:
   - `<ProjectName>/wiki/sources/.gitkeep` (empty note)
   - `<ProjectName>/wiki/entities/.gitkeep`
   - `<ProjectName>/wiki/concepts/.gitkeep`
   - `<ProjectName>/wiki/decisions/.gitkeep`
   - `<ProjectName>/wiki/learnings/.gitkeep`
   - `<ProjectName>/wiki/queries/.gitkeep`
   - `<ProjectName>/raw/articles/.gitkeep`
   - `<ProjectName>/raw/docs/.gitkeep`
   - `<ProjectName>/raw/notes/.gitkeep`
   - `<ProjectName>/raw/assets/.gitkeep`

### Step 8: Update CLAUDE.md

Append the wiki rules section to the project's CLAUDE.md (create it if it doesn't exist).
If a CLAUDE.md already exists, add a new section without removing existing content.

The content to append is specified in the **CLAUDE.md Wiki Rules** section below.

### Step 9: Confirm to User

Report what was created:
- Number of directories created locally
- Number of notes created in Obsidian
- Confirm CLAUDE.md was updated
- Remind the user of available commands

---

## CLAUDE.md Wiki Rules

This is the content to inject into the project's CLAUDE.md during bootstrap.
It ensures Claude Code follows the wiki protocol in ALL future sessions.

```markdown
## Wiki - Persistent Knowledge Base

This project uses **Claude Wiki** (based on the LLM Wiki methodology by Andrej Karpathy)
to maintain a persistent, cumulative knowledge base. You (Claude Code) are the exclusive
maintainer of this wiki. Follow these rules rigorously in EVERY session.

### Architecture

Three layers:
1. **raw/** — Immutable sources. You read but NEVER modify.
2. **wiki/** — Pages you generate and maintain. You own this layer.
3. **CLAUDE.md** (this file) — Schema defining wiki behavior.

Dual-write: every wiki write goes to BOTH local files AND Obsidian vault via MCP.
Obsidian path: `<ProjectName>/wiki/` and `<ProjectName>/raw/`.

### Page Format

Every wiki page MUST have this YAML frontmatter:

    ---
    title: "Page Name"
    type: source | entity | concept | decision | learning | query
    created: YYYY-MM-DD
    updated: YYYY-MM-DD
    tags: [tag1, tag2]
    sources: [related source files]
    related: [related wiki pages]
    ---

Use wikilinks `[[page-name]]` for cross-references.
File names in kebab-case: `my-concept.md`.
Write pages in the same language the user uses in conversation.

### Session Start Protocol

At the START of every session, BEFORE doing anything else:
1. Read `wiki/index.md` to understand wiki state
2. Read `wiki/log.md` to see recent operations
3. Check for new files in `raw/` that haven't been ingested

### Operations

#### INGEST — Process a source

When the user asks to ingest a source from `raw/`:

1. Read the full source content
2. Discuss key points with the user
3. Create summary page in `wiki/sources/<source-name>.md`
4. Create or UPDATE entity pages in `wiki/entities/` for each technology, lib, service mentioned
5. Create or UPDATE concept pages in `wiki/concepts/` for each pattern, architecture, methodology
6. UPDATE `wiki/index.md` with new pages
7. APPEND entry to `wiki/log.md`
8. SYNC all created/updated pages to Obsidian using `mcp__obsidian__write_note`
9. Report to user: pages created, pages updated, entities extracted

#### QUERY — Search the wiki

When the user asks a question:

1. Read `wiki/index.md` to find relevant pages
2. Also search Obsidian: `mcp__obsidian__search_notes` for broader matches
3. Read relevant pages and synthesize answer with citations
4. If the answer is valuable (analysis, comparison, insight), ask user if it should be saved to `wiki/queries/`
5. If yes, create page and update index (both local and Obsidian)

#### LINT — Health check

When the user asks for lint/maintenance:

- Check for contradictions between pages
- Find orphan pages (no incoming links)
- Find mentioned concepts without their own page
- Check for missing cross-references
- Identify stale information
- Suggest new sources or investigations
- Log the lint result

#### DECISION — Record a decision

When an important decision is made:

1. Create page in `wiki/decisions/<decision-name>.md` with:
   - Context: what problem was being solved
   - Options considered
   - Decision taken and justification
   - Trade-offs accepted
   - Links to related concepts and entities
2. Update index and log (local + Obsidian)

#### LEARNING — Record a lesson

When a bug is fixed, an insight emerges, or something important is learned:

1. Create page in `wiki/learnings/<learning-name>.md` with:
   - What happened
   - Root cause
   - Solution applied
   - How to prevent in the future
   - Links to related concepts and entities
2. Update index and log (local + Obsidian)

#### SYNC — Synchronize with Obsidian

When the user asks to sync, or after any batch operation:

1. Read all local wiki pages
2. For each page, use `mcp__obsidian__write_note` to write/update in Obsidian
3. Use `mcp__obsidian__manage_tags` to ensure tags are consistent
4. Log the sync operation

### Automatic Behaviors

- **Bug fixed?** Ask if it should be recorded as a learning
- **Architecture decision?** Ask if it should be recorded as a decision
- **Valuable analytical response?** Ask if it should be archived as a query
- **Contradiction with existing wiki knowledge?** Alert the user
- **New file in raw/?** Ask if it should be ingested

### Quick Commands

- `wiki ingest <file>` — Ingest specific source
- `wiki ingest all` — Ingest all new sources in raw/
- `wiki query <question>` — Query the wiki
- `wiki lint` — Run health check
- `wiki status` — Show stats (pages, sources, last update)
- `wiki decision <title>` — Record a decision
- `wiki learning <title>` — Record a lesson
- `wiki search <term>` — Search wiki pages
- `wiki sync` — Force sync local → Obsidian
```

---

## Operations Reference (for skill executor)

This section provides detailed guidance for each operation the skill supports.

### Dual-Write Protocol

Every time you create or update a wiki page, you MUST:

1. **Write locally** using the Write or Edit tool to the `wiki/` directory
2. **Write to Obsidian** using `mcp__obsidian__write_note` with path `<ProjectName>/wiki/<subdir>/<filename>.md`
3. **Update frontmatter in Obsidian** using `mcp__obsidian__update_frontmatter` if only metadata changed
4. **Manage tags** using `mcp__obsidian__manage_tags` to keep Obsidian tags in sync

If Obsidian MCP is unavailable (connection error or not installed), write locally and log a warning.
The user can run `wiki sync` later to push local changes to Obsidian.

If the user hasn't set up the MCP yet and asks for any wiki operation, remind them:

> Obsidian sync is not active. Want me to set up the MCP server? It takes 30 seconds.
> Or you can run in local-only mode — your wiki will work fine, just without Obsidian mirroring.

### Ingest Workflow (Detailed)

When ingesting a source:

1. Read the source file from `raw/`
2. Extract:
   - **Key points**: Main arguments, findings, recommendations
   - **Entities**: Technologies, libraries, APIs, services, people, organizations
   - **Concepts**: Patterns, architectures, methodologies, principles
   - **Quotes**: Notable direct quotes worth preserving
3. Create `wiki/sources/<source-name>.md` with structured summary
4. For each entity: check if `wiki/entities/<entity>.md` exists
   - If yes: update it with new information from this source
   - If no: create it with what's known from this source
5. For each concept: same as entities but in `wiki/concepts/`
6. Update `wiki/index.md` — add new entries under appropriate sections
7. Append to `wiki/log.md`: `## [<TODAY>] ingest | <source-name>`
8. Dual-write all changes to Obsidian

### Query Workflow (Detailed)

1. Read `wiki/index.md`
2. Use `mcp__obsidian__search_notes` with the query for broader search
3. Read the most relevant pages (usually 2-5)
4. Synthesize answer citing wiki pages: "According to [[page-name]], ..."
5. If the user says to save, create `wiki/queries/<query-slug>.md`

### Sync Workflow (Detailed)

1. List all `.md` files in `wiki/` recursively (use Glob `wiki/**/*.md`)
2. For each file:
   - Read local content
   - Use `mcp__obsidian__write_note` with path `<ProjectName>/wiki/<relative-path>`
   - Use `mcp__obsidian__update_frontmatter` to ensure metadata is current
3. Check for files in Obsidian that don't exist locally (using `mcp__obsidian__list_directory`)
4. Log: `## [<TODAY>] sync | Full sync to Obsidian (<N> pages)`

### Status Command

When user runs `wiki status`, report:
- Total pages by type (sources, entities, concepts, decisions, learnings, queries)
- Last ingest date
- Last lint date
- Total sources in raw/ (ingested vs pending)
- Obsidian sync status

---

## Caveats and Best Practices

From Karpathy's methodology and community feedback:

1. **Ground in sources**: Always cite the original source when synthesizing. The wiki is a navigation layer, not a replacement for raw sources.
2. **Error accumulation**: Incorrect information can compound. When updating existing pages, re-read the original source to verify claims.
3. **Information loss**: Summaries compress detail. Preserve notable quotes and specific data points.
4. **Prefer many small pages**: A page per entity, per concept, per decision. Interlink heavily.
5. **The user is the curator**: You maintain, but the user directs. Always discuss key points during ingest rather than silently filing everything away.
6. **Hybrid approach**: Use the wiki for navigation and synthesis, but always be able to point back to the original source in `raw/`.

---

## Credits

Methodology based on the **LLM Wiki** article by **Andrej Karpathy** — a pattern for building
personal knowledge bases with LLMs, where the agent builds and maintains a persistent wiki of
interlinked markdown files instead of rediscovering knowledge from scratch each session.

Original article: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

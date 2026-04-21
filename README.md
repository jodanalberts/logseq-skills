# Logseq Skills

Agent Skills for use with [Logseq](https://logseq.com/).

These skills follow the [Agent Skills specification](https://agentskills.io/specification) so they can be used by any skills-compatible agent, including Claude Code.

## Installation

### Claude Code

Add the contents of this repo to a `/.claude` folder in the root of your Logseq graph (or whichever folder you're using with Claude Code). See more in the [official Claude Skills documentation](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview).

### npx skills

```
npx skills add git@github.com:jalberts/logseq-skills.git
```

## Skills

| Skill | Description |
|-------|-------------|
| [logseq-markdown](skills/logseq-markdown) | Create and edit Logseq Flavored Markdown (`.md`) with block-based structure, page links, block references, embeds, properties, tasks, and queries |
| [logseq-queries](skills/logseq-queries) | Build Logseq simple queries and advanced Datalog queries to filter and display blocks across your graph |
| [logseq-api](skills/logseq-api) | Interact with a running Logseq instance via the local HTTP API to read, create, update, and search pages and blocks |

## Key Concepts

Logseq differs from Obsidian in several fundamental ways:

- **Block-based**: All content is organized in a hierarchy of bullet-point blocks, not free-form paragraphs
- **Outliner-first**: Every piece of content is a block; nesting creates hierarchy, not just visual indentation
- **Block references**: Any block can be referenced by its UUID with `((uuid))` syntax
- **Inline properties**: Properties use `key:: value` syntax inline in a block, not YAML frontmatter
- **Built-in task management**: Tasks use org-mode-inspired markers (`TODO`, `DOING`, `DONE`, etc.)
- **Bidirectional links**: `[[Page Name]]` creates a page and links to it simultaneously

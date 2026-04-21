# Logseq Properties Reference

Logseq uses `key:: value` inline syntax for metadata. Unlike Obsidian's YAML frontmatter, properties live inside blocks.

## Syntax

```markdown
- key:: value
  another-key:: another value
```

Property lines must be indented under their block when on a separate line, or can each start with `- ` as their own block (for page-level properties).

## Page-Level Properties

The **first block** of a page file becomes page-level metadata when it contains only properties (no other content). These properties appear in the page info panel and are queryable.

```markdown
- title:: My Custom Title
  tags:: project, active
  author:: [[Alice]]
  status:: in-progress
  created:: [[2024-01-15]]
  url:: https://example.com
```

> If you want the first block to have visible content AND properties, put the properties after a line of content. In that case, properties become block-level metadata, not page-level.

## Block-Level Properties

Properties on any non-first block, or on first blocks that also have content:

```markdown
- My block content
  status:: done
  priority:: high
  due:: [[2024-01-20]]
```

## Built-in Properties

| Property | Description |
|----------|-------------|
| `title::` | Set a display title distinct from the filename (page-level) |
| `tags::` | Comma-separated list of tags/pages |
| `alias::` | Alternative names that resolve to this page when linked (e.g. `alias:: Alt Name, Other Name`) |
| `icon::` | Emoji or icon for the page |
| `template::` | Mark a block or page as a template |
| `template-including-parent::` | Whether to include parent when using template |
| `id::` | Block UUID (auto-assigned, do not set manually) |
| `collapsed::` | Whether a block is collapsed (`true` or `false`) |
| `heading::` | Render block as heading level (1–6) |
| `background-color::` | Block background color |
| `background-image::` | Block background image URL |
| `ls-type::` | Internal Logseq type (e.g. `asset`) |

## Value Types

### Plain Text

```markdown
- status:: in-progress
  author:: Alice Smith
```

### Page References

Link to another page as a property value:

```markdown
- project:: [[Project Alpha]]
  assigned-to:: [[Alice]]
```

### Date (Journal Page Reference)

Use a journal page link for date values:

```markdown
- due:: [[2024-01-20]]
  created:: [[2024-01-01]]
```

### Tags (Comma-separated)

```markdown
- tags:: project, active, frontend
```

Tags can also be page references:

```markdown
- tags:: [[Project Alpha]], [[Frontend]], active
```

### Numbers

```markdown
- priority:: 1
  effort:: 3
  word-count:: 1500
```

### Booleans

```markdown
- done:: true
  archived:: false
```

### URLs

```markdown
- url:: https://example.com
  source:: https://arxiv.org/abs/1234.56789
```

## Multiple Values

Comma-separate multiple values for list-type properties:

```markdown
- tags:: project, active, frontend
  related:: [[Page A]], [[Page B]], [[Page C]]
```

## Querying Properties

Properties are queryable in both simple and advanced queries:

```markdown
{{query (property :status "done")}}
{{query (property :tags "project")}}
{{query (and (property :status "in-progress") (property :priority 1))}}
```

In advanced queries, access properties via `:block/properties`:

```clojure
[?b :block/properties ?props]
[(get ?props :status) ?status]
[(= ?status "done")]
```

## Namespace Conventions

Common property naming patterns used in the community:

| Property | Typical Use |
|----------|-------------|
| `tags::` | Category/type labels |
| `status::` | Workflow state (e.g. `todo`, `doing`, `done`) |
| `priority::` | Numeric priority (1 = highest) |
| `due::` | Due date as `[[YYYY-MM-DD]]` |
| `created::` | Creation date as `[[YYYY-MM-DD]]` |
| `author::` | Author as `[[Name]]` |
| `source::` | URL or reference origin |
| `related::` | Related pages |
| `alias::` | Alternative page names |
| `type::` | Note type (e.g. `book`, `person`, `project`) |

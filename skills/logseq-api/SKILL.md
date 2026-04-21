---
name: logseq-api
description: Interact with a running Logseq instance via the local HTTP API to read, create, update, search, and manage pages and blocks programmatically. Use when the user wants to automate Logseq operations, query the graph from outside the app, build integrations, or interact with their Logseq graph from the command line.
---

# Logseq HTTP API Skill

Use the Logseq HTTP API to interact with a running Logseq desktop instance. This is the programmatic equivalent of using Logseq interactively — requires Logseq to be open.

## Setup

1. Open Logseq → **Settings** → **Developer** → **Enable HTTP APIs server**
2. Set an **Authorization token** in the same panel
3. The server runs at `http://localhost:12315` by default

## Authentication

All requests require the `Authorization` header:

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" ...
```

Set your token in an environment variable to avoid repeating it:

```bash
export LOGSEQ_TOKEN="your-token-here"
export LOGSEQ_API="http://localhost:12315/api"
```

## API Structure

All calls use a **single endpoint** `POST /api` with a `method` field that maps to the Logseq Plugin SDK and an `args` array of positional arguments:

```bash
curl -s -X POST $LOGSEQ_API \
  -H "Authorization: Bearer $LOGSEQ_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"method": "logseq.Editor.getPage", "args": ["My Page"]}'
```

Methods follow the Plugin SDK namespace: `logseq.Editor.*`, `logseq.DB.*`, `logseq.App.*`.

## Common Patterns

```bash
# Check API is responding
curl -s -X POST $LOGSEQ_API \
  -H "Authorization: Bearer $LOGSEQ_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"method": "logseq.App.getCurrentGraph", "args": []}'

# Get a page by name
curl -s -X POST $LOGSEQ_API \
  -H "Authorization: Bearer $LOGSEQ_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"method": "logseq.Editor.getPage", "args": ["My Page"]}'

# Full-text search
curl -s -X POST $LOGSEQ_API \
  -H "Authorization: Bearer $LOGSEQ_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"method": "logseq.DB.q", "args": ["[:find (pull ?b [:block/content]) :where [?b :block/content ?c] [(clojure.string/includes? ?c \"search term\")]]"]}'
```

## Method Reference

### App / Graph

| Method | Args | Description |
|--------|------|-------------|
| `logseq.App.getCurrentGraph` | `[]` | Info about the open graph |
| `logseq.App.showMsg` | `[content, status?]` | Show a notification (`"success"`, `"warning"`, `"error"`) |

### Pages

| Method | Args | Description |
|--------|------|-------------|
| `logseq.Editor.getAllPages` | `[]` | All pages in the graph |
| `logseq.Editor.getPage` | `[name]` | Get a page by name |
| `logseq.Editor.getPageBlocksTree` | `[name]` | Get page blocks as a nested tree |
| `logseq.Editor.createPage` | `[name, properties?, opts?]` | Create a new page |

`createPage` opts: `{"createFirstBlock": false, "redirect": false, "journal": false}`

### Blocks

| Method | Args | Description |
|--------|------|-------------|
| `logseq.Editor.getBlock` | `[uuid]` | Get a block by UUID |
| `logseq.Editor.insertBlock` | `[srcBlock, content, opts?]` | Insert a block relative to `srcBlock` |
| `logseq.Editor.appendBlockInPage` | `[page, content]` | Append a block to the end of a page |
| `logseq.Editor.updateBlock` | `[uuid, content]` | Update a block's content |
| `logseq.Editor.removeBlock` | `[uuid]` | Delete a block |
| `logseq.Editor.moveBlock` | `[srcId, targetId, opts?]` | Move a block |

`insertBlock` opts: `{"sibling": false, "before": false, "focus": false}`

### Database Queries

| Method | Args | Description |
|--------|------|-------------|
| `logseq.DB.q` | `[datalogQuery]` | Run a Datalog query string |
| `logseq.DB.datascriptQuery` | `[query, ...inputs]` | Run a parameterized Datalog query |

## Scripting Examples

### Get All TODO Tasks

```bash
curl -s -X POST $LOGSEQ_API \
  -H "Authorization: Bearer $LOGSEQ_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"method": "logseq.DB.q", "args": ["[:find (pull ?b [:block/content :block/scheduled]) :where [?b :block/marker \"TODO\"]]"]}' \
  | jq '.result'
```

### Append to Today's Journal

```bash
TODAY=$(date +"%Y_%m_%d")
curl -s -X POST $LOGSEQ_API \
  -H "Authorization: Bearer $LOGSEQ_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"method\": \"logseq.Editor.appendBlockInPage\", \"args\": [\"$TODAY\", \"Added via HTTP API at $(date)\"]}"
```

### Create a Page with Properties

```bash
curl -s -X POST $LOGSEQ_API \
  -H "Authorization: Bearer $LOGSEQ_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "method": "logseq.Editor.createPage",
    "args": [
      "New Meeting Notes",
      {"tags": "meeting, notes", "date": "[[2024-01-15]]"},
      {"createFirstBlock": false}
    ]
  }'
```

### Insert a Child Block

```bash
curl -s -X POST $LOGSEQ_API \
  -H "Authorization: Bearer $LOGSEQ_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "method": "logseq.Editor.insertBlock",
    "args": [
      "parent-block-uuid-here",
      "TODO [#A] New child task",
      {"sibling": false}
    ]
  }'
```

### Get All Blocks in a Page as Tree

```bash
PAGE="My Project"
curl -s -X POST $LOGSEQ_API \
  -H "Authorization: Bearer $LOGSEQ_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"method\": \"logseq.Editor.getPageBlocksTree\", \"args\": [\"$PAGE\"]}" \
  | jq '.'
```

### Show a Notification

```bash
curl -s -X POST $LOGSEQ_API \
  -H "Authorization: Bearer $LOGSEQ_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"method": "logseq.App.showMsg", "args": ["Script complete!", "success"]}'
```

## Response Format

Successful responses return a `result` key:

```json
{"result": { ... }}
```

Error responses:

```json
{"error": "Description of the error"}
```

## Working with Block Content

Block content in API calls uses plain Logseq markdown — no leading `- ` bullet (the API adds the block structure). For properties on an inserted block, append them on new lines with `\n`:

```bash
-d '{"method": "logseq.Editor.insertBlock", "args": ["parent-uuid", "TODO [#A] Fix the bug\nSCHEDULED: <2024-01-20 Sat>", {}]}'
```

## Plugin Development

For more complex automation, the Logseq Plugin SDK provides the same methods plus event listeners, UI injection, and real-time graph access:

- [Logseq Plugin API Docs](https://logseq.github.io/plugins/)
- [Plugin SDK](https://github.com/logseq/logseq-plugin-sdk)

## Troubleshooting

**Connection refused**: Verify the HTTP server is enabled in Logseq Settings → Developer → Enable HTTP APIs server, and that Logseq is running.

**401 Unauthorized**: Check that the `Authorization: Bearer TOKEN` header matches the token set in Logseq settings exactly.

**Empty results from `logseq.DB.q`**: Logseq's Datalog schema uses lowercase page names (`:block/name` is always lowercase). Task markers must be uppercase strings: `"TODO"` not `"todo"`.

**Method not found**: Verify the method name matches the Plugin SDK exactly — e.g. `logseq.Editor.getPage` not `logseq.editor.getPage`.

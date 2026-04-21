# Logseq Tasks Reference

Logseq has a built-in task management system inspired by org-mode. Task markers appear at the start of a block's content.

## Two Workflows

Logseq's `:preferred-workflow` setting in `logseq/config.edn` determines which markers the keyboard shortcut cycles through:

| Setting | Cycle |
|---------|-------|
| `:todo` (default) | `TODO` → `DOING` → `DONE` |
| `:now` | `LATER` → `NOW` → `DONE` |

**Pick one workflow and use its markers consistently.** Both sets of markers are technically valid in any graph, but mixing them makes queries and the agenda unreliable. The shortcut (`cmd/ctrl+enter` on a block) only cycles through the markers in your configured workflow.

## Task Markers

**`:todo` workflow** (default):

| Marker | Meaning |
|--------|---------|
| `TODO` | Not yet started |
| `DOING` | Actively in progress |
| `DONE` | Completed |

**`:now` workflow**:

| Marker | Meaning |
|--------|---------|
| `LATER` | Deferred, low priority |
| `NOW` | Actively in progress right now |
| `DONE` | Completed |

**Available in both workflows**:

| Marker | Meaning |
|--------|---------|
| `WAITING` | Blocked on someone or something |
| `WAIT` | Alias for WAITING |
| `CANCELLED` | Won't be done |
| `CANCELED` | Alias for CANCELLED |

## Syntax

```markdown
- TODO Task description
- DOING Task in progress
- DONE Completed task
- WAITING Blocked on external dependency
- CANCELLED Abandoned task
```

## Priority Markers

Add `[#A]`, `[#B]`, or `[#C]` after the task marker:

```markdown
- TODO [#A] Critical — must ship today
- TODO [#B] Important but not urgent
- TODO [#C] Nice to have
- DOING [#A] Fixing the production incident
```

## Scheduled and Deadline Dates

```markdown
- TODO Write the quarterly report
  SCHEDULED: <2024-01-20 Sat>

- TODO Submit tax forms
  DEADLINE: <2024-04-15 Mon>

- TODO Weekly team sync
  SCHEDULED: <2024-01-15 Mon .+1w>
```

**Date formats**:

| Format | Description |
|--------|-------------|
| `<2024-01-15 Mon>` | Specific date |
| `<2024-01-15 Mon 09:00>` | Specific date and time |
| `<2024-01-15 Mon .+1w>` | Repeat every 1 week from completion |
| `<2024-01-15 Mon +1w>` | Repeat every 1 week from scheduled date |
| `<2024-01-15 Mon ++1w>` | Repeat every 1 week, catch up missed occurrences |

**Repeat interval units**: `h` (hour), `d` (day), `w` (week), `m` (month), `y` (year)

## Logbook (Time Tracking)

Logseq records clock-in/clock-out in a logbook when you use the built-in timer:

```markdown
- DONE Write the report
  :LOGBOOK:
  CLOCK: [2024-01-14 Sun 09:00]--[2024-01-14 Sun 11:00] =>  2:00
  CLOCK: [2024-01-15 Mon 13:00]--[2024-01-15 Mon 14:30] =>  1:30
  :END:
```

Total time is the sum of all CLOCK entries. Do not edit logbook entries manually unless correcting errors.

## Querying Tasks

Use simple queries to surface tasks across the graph:

```markdown
{{query (task TODO)}}
{{query (task TODO DOING NOW)}}
{{query (and (task TODO) (page [[Project Alpha]]))}}
{{query (and (task TODO) (priority A))}}
{{query (and (task TODO) (between [[2024-01-01]] [[2024-01-31]]))}}
```

**Query all unfinished tasks by priority**:

```markdown
{{query (and (task TODO DOING NOW LATER WAIT) (priority A))}}
```

**Query overdue tasks** (scheduled before today):

Use an advanced query — see `logseq-queries` skill.

## Task Workflow Patterns

### Daily Journal Tasks

Add tasks to the current journal page and they appear in the agenda:

```markdown
- NOW [#A] Fix the login bug
  SCHEDULED: <2024-01-15 Mon>
- TODO Review Alice's pull request
- LATER Read the new RFC
```

### Project Task List

In a project page, group tasks by section:

```markdown
- ## Active
	- DOING [#A] Build authentication flow
	- DOING Write API documentation

- ## Queued
	- TODO [#B] Implement search feature
	- TODO [#C] Add dark mode

- ## Done
	- DONE Project setup and repository
	- DONE Initial wireframes
```

### Linked Tasks

Reference source material inside a task:

```markdown
- TODO Review feedback from ((6626d597-5ecf-4a9e-bd57-7dde55e1cecd))
- TODO Implement design from [[UI Spec v2#Button styles]]
```

## Workflow State Machines

**`:todo` workflow**:
- Basic: `TODO` → `DOING` → `DONE`
- With blocking: `TODO` → `WAITING` → `TODO` → `DOING` → `DONE`
- Abandoned: any state → `CANCELLED`

**`:now` workflow**:
- Basic: `LATER` → `NOW` → `DONE`
- With blocking: `LATER` → `WAITING` → `LATER` → `NOW` → `DONE`
- Abandoned: any state → `CANCELLED`

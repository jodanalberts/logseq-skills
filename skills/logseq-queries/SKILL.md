---
name: logseq-queries
description: Build Logseq simple queries and advanced Datalog queries to filter and display blocks across your graph. Use when creating {{query}} blocks, #+BEGIN_QUERY blocks, filtering tasks, searching by property or tag, or when the user wants to create database-like views of their notes in Logseq.
---

# Logseq Queries Skill

Logseq has two query systems: **simple queries** for common filtering and **advanced queries** written in Datalog (Clojure EDN) for complex logic.

## Workflow: Creating a Query

1. **Choose type**: Use simple queries for common patterns; advanced queries for complex logic, sorting, or transformations
2. **Write the query** using the syntax below
3. **Place it in a block**: Simple queries inline with `{{query ...}}`, advanced queries in `#+BEGIN_QUERY ... #+END_QUERY` blocks. Type `/query` in any block to insert an advanced query template with the Logseq editor UI.
4. **Validate**: Run the query in Logseq and confirm the results match expectations. Check for syntax errors in the developer console if results are empty unexpectedly

---

## Simple Queries

Simple queries use a Lisp-like DSL inside `{{query ...}}` blocks.

### Basic Syntax

```markdown
{{query filter-expression}}
{{query (combinator filter1 filter2 ...)}}
```

### Task Filters

```markdown
{{query (task TODO)}}
{{query (task DONE)}}
{{query (task TODO DOING NOW)}}
{{query (task TODO LATER WAIT)}}
```

Valid task markers: `TODO`, `DOING`, `NOW`, `LATER`, `WAIT`, `WAITING`, `DONE`, `CANCELLED`, `CANCELED`

### Priority Filter

```markdown
{{query (priority A)}}
{{query (priority B)}}
{{query (priority A B)}}
```

### Page Filter

Return all blocks that appear in a specific page:

```markdown
{{query (page [[My Project]])}}
{{query (page "exact-page-name")}}
```

### Tag / Page-tag Filter

Return pages that have a given tag:

```markdown
{{query (page-tags #project)}}
{{query (page-tags [[book]])}}
```

### Property Filter

Return blocks that have a property with a specific value:

```markdown
{{query (property :status "done")}}
{{query (property :priority 1)}}
{{query (property :tags "project")}}
```

Return blocks that have a property with any value:

```markdown
{{query (property :due)}}
```

### Date / Between Filter

Return blocks scheduled or created between two journal dates:

```markdown
{{query (between [[2024-01-01]] [[2024-01-31]])}}
{{query (between -7d today)}}
{{query (between today +7d)}}
```

Date shortcuts: `today`, `yesterday`, `tomorrow`, `-Nd` (N days ago), `+Nd` (N days from now)

### Namespace Filter

Return all pages within a namespace:

```markdown
{{query (namespace [[Projects]])}}
```

### Full-Text Search

```markdown
{{query (full-text-search "keyword")}}
```

### Combinators

```markdown
{{query (and filter1 filter2)}}
{{query (or filter1 filter2)}}
{{query (not filter)}}
```

Nested combinators:

```markdown
{{query (and
  (task TODO)
  (or
    (page [[Project Alpha]])
    (page [[Project Beta]])))}}
```

### Common Patterns

**All active tasks for a project**:

```markdown
{{query (and (task TODO DOING NOW) (page [[My Project]]))}}
```

**High-priority unfinished tasks**:

```markdown
{{query (and (task TODO DOING NOW LATER) (priority A))}}
```

**All pages tagged as books**:

```markdown
{{query (page-tags #book)}}
```

**Tasks due this week**:

```markdown
{{query (and (task TODO) (between today +7d))}}
```

**Blocks with a property set**:

```markdown
{{query (property :status)}}
```

**Tasks with specific status property**:

```markdown
{{query (and (task TODO) (property :project "Alpha"))}}
```

---

## Advanced Queries

Advanced queries use Datalog syntax written in Clojure EDN. They are placed inside `#+BEGIN_QUERY ... #+END_QUERY` blocks and support sorting, transformations, grouped display, and any logic expressible in Datalog.

### Block Syntax

````markdown
#+BEGIN_QUERY
{:title "My Query Title"
 :query [:find (pull ?b [*])
         :where
         [?b :block/marker "TODO"]]
 :collapsed? true}
#+END_QUERY
````

### Options

| Key | Type | Description |
|-----|------|-------------|
| `:title` | string | Display title for the query block |
| `:query` | Datalog | The query vector (required) |
| `:inputs` | vector | Input variables, e.g. `[:today]` |
| `:result-transform` | fn | Clojure function to transform results |
| `:collapsed?` | bool | Collapse query block by default |
| `:breadcrumb-show?` | bool | Show breadcrumb path for each result |
| `:table-view?` | bool | Display results as a table |
| `:group-by-page?` | bool | Group results by their source page |

### Datalog Basics

Datalog queries use entity-attribute-value triplets. Each clause `[?entity :attribute ?value]` constrains results.

**Core block attributes**:

| Attribute | Type | Description |
|-----------|------|-------------|
| `:block/content` | string | Raw text content of the block |
| `:block/marker` | string | Task marker (`"TODO"`, `"DONE"`, etc.) |
| `:block/priority` | string | Priority (`"A"`, `"B"`, `"C"`) |
| `:block/page` | ref | Page entity the block belongs to |
| `:block/scheduled` | int | Scheduled date as integer (`YYYYMMDD`) |
| `:block/deadline` | int | Deadline date as integer (`YYYYMMDD`) |
| `:block/properties` | map | Block properties map |
| `:block/tags` | refs | Tags on the block |
| `:block/uuid` | uuid | Unique block identifier |
| `:block/refs` | refs | Pages/blocks referenced in this block |
| `:block/parent` | ref | Parent block entity |
| `:block/left` | ref | Sibling ordering (left neighbor) |
| `:block/level` | int | Nesting depth |

**Core page attributes**:

| Attribute | Type | Description |
|-----------|------|-------------|
| `:block/name` | string | Lowercase page name |
| `:block/original-name` | string | Original-case page name |
| `:block/tags` | refs | Page tags |
| `:block/properties` | map | Page properties map |
| `:block/file` | ref | File entity for the page |
| `:block/journal?` | bool | Whether this is a journal page |
| `:block/journal-day` | int | Journal date as integer (`YYYYMMDD`) |

### Query Examples

**All TODO tasks**:

```clojure
#+BEGIN_QUERY
{:title "All TODOs"
 :query [:find (pull ?b [*])
         :where
         [?b :block/marker "TODO"]]
 :collapsed? false}
#+END_QUERY
```

**Tasks with a specific property value**:

```clojure
#+BEGIN_QUERY
{:title "In-progress items"
 :query [:find (pull ?b [*])
         :where
         [?b :block/properties ?props]
         [(get ?props :status) ?status]
         [(= ?status "in-progress")]]}
#+END_QUERY
```

**All blocks linking to a specific page**:

```clojure
#+BEGIN_QUERY
{:title "Mentions of Project Alpha"
 :query [:find (pull ?b [*])
         :where
         [?p :block/name "project alpha"]
         [?b :block/refs ?p]]}
#+END_QUERY
```

**Tasks scheduled before today (overdue)**:

```clojure
#+BEGIN_QUERY
{:title "Overdue Tasks"
 :query [:find (pull ?b [*])
         :in $ ?today
         :where
         [?b :block/marker ?marker]
         [(contains? #{"TODO" "DOING" "NOW" "WAIT"} ?marker)]
         [?b :block/scheduled ?scheduled]
         [(< ?scheduled ?today)]]
 :inputs [:today]}
#+END_QUERY
```

**Journal pages from the last 7 days**:

```clojure
#+BEGIN_QUERY
{:title "Last 7 Days"
 :query [:find (pull ?p [:block/original-name :block/journal-day])
         :in $ ?start
         :where
         [?p :block/journal? true]
         [?p :block/journal-day ?day]
         [(>= ?day ?start)]]
 :inputs [:7d-before]
 :result-transform (fn [result]
                     (sort-by (fn [p] (get p :block/journal-day)) #(compare %2 %1) result))}
#+END_QUERY
```

**Pages tagged as books, sorted by title**:

```clojure
#+BEGIN_QUERY
{:title "Book List"
 :query [:find (pull ?p [:block/original-name :block/properties])
         :where
         [?tag :block/name "book"]
         [?p :block/tags ?tag]]
 :result-transform (fn [result]
                     (sort-by (fn [p] (get p :block/original-name)) result))}
#+END_QUERY
```

**Blocks with property, displayed as table**:

```clojure
#+BEGIN_QUERY
{:title "Project Status"
 :query [:find (pull ?b [*])
         :where
         [?b :block/properties ?props]
         [(get ?props :status) ?status]]
 :table-view? true
 :breadcrumb-show? false}
#+END_QUERY
```

### `:inputs` Built-in Values

| Input | Description |
|-------|-------------|
| `:today` | Today's date as integer (`YYYYMMDD`) |
| `:yesterday` | Yesterday's date |
| `:tomorrow` | Tomorrow's date |
| `:1d-before` | 1 day ago (also `:7d-before`, `:14d-before`, `:30d-before`) |
| `:1d-after` | 1 day ahead (also `:7d-after`, `:14d-after`, `:30d-after`) |
| `:start-of-today-week` | Start of the current week |
| `:end-of-today-week` | End of the current week |
| `:current-page` | Name of the page where the query lives (string) |

### `:result-transform` Patterns

Result transforms are Clojure functions that operate on the raw query results:

```clojure
; Sort by scheduled date
:result-transform (fn [result]
                   (sort-by (fn [b] (get b :block/scheduled)) result))

; Sort by priority (A before B before C)
:result-transform (fn [result]
                   (sort-by (fn [b] (get b :block/priority "Z")) result))

; Filter results further
:result-transform (fn [result]
                   (filter (fn [b] (some? (get b :block/scheduled))) result))

; Limit results
:result-transform (fn [result]
                   (take 10 result))

; Sort descending by journal day
:result-transform (fn [result]
                   (sort-by (fn [b]
                              (get-in b [:block/page :block/journal-day] 0))
                            #(compare %2 %1)
                            result))
```

## Troubleshooting

**Query returns no results**:
- Check page names are lowercase in queries (`:block/name` is always lowercase)
- Verify task markers are uppercase strings: `"TODO"` not `"todo"`
- Journal day integers are `YYYYMMDD` (e.g., `20240115`), not timestamps

**Simple query syntax error**:
- Every opening `(` needs a matching `)`
- Check that page references use `[[double brackets]]` not single

**Advanced query empty pull result**:
- Use `(pull ?b [*])` to pull all attributes for debugging
- Narrow the `:where` clauses one at a time to find which clause eliminates results

## References

- [Logseq Queries Documentation](https://docs.logseq.com/#/page/queries)
- [Logseq Advanced Queries](https://docs.logseq.com/#/page/advanced%20queries)
- [Datalog Tutorial](https://docs.datomic.com/pro/query/query.html)

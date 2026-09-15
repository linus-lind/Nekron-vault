---
type: feature
source: "[[<% tp.system.prompt('Data source note') %>]]"
status: active        # active | experimental | deprecated
created: <% tp.date.now("YYYY-MM-DD") %>
tags: [feature]
---

# <% tp.file.title %>

## Definition
Exact formula / computation. Point to the code function that produces it.
`Nekron/features/...`

## Rationale
What does this feature capture?

## Leakage check
Is every input strictly available at prediction time? Any forward-fill risk?

## Used by models
```dataview
LIST FROM "04_Models" WHERE contains(file.outlinks, this.file.link)
```

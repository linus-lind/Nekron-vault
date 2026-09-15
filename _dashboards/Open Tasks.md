---
type: dashboard
tags: [dashboard]
---

# Open Tasks (across the whole vault)

Requires the **Tasks** plugin (or Dataview's task query below).

## All incomplete tasks
```dataview
TASK
WHERE !completed
GROUP BY file.link
```

## With due dates
```dataview
TASK
WHERE !completed AND due
SORT due ASC
```

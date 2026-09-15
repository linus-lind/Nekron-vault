---
type: daily
created: <% tp.date.now("YYYY-MM-DD") %>
tags: [daily]
---

# <% tp.date.now("dddd, MMMM D, YYYY") %>

## Focus today
- 

## Work log
- 

## Tasks
- [ ] 

## Experiments touched today
```dataview
LIST FROM "05_Experiments" WHERE created = date("<% tp.date.now("YYYY-MM-DD") %>")
```

## Notes & ideas
Anything to capture → send to [[00_Inbox]] or spin a note.

## Tomorrow
- 

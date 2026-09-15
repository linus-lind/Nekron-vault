---
type: dashboard
tags: [dashboard]
---

# Research Reading

## To read (by relevance)
```dataview
TABLE authors, year, rating, url
FROM "02_Research/Papers"
WHERE status = "to-read"
SORT rating DESC
```

## Reading now
```dataview
LIST FROM "02_Research/Papers" WHERE status = "reading"
```

## Read — high relevance (≥4/5)
```dataview
TABLE rating, year FROM "02_Research/Papers"
WHERE status = "read" AND rating >= 4
SORT rating DESC
```

## Concept library
```dataview
LIST FROM "02_Research/Concepts" SORT file.name ASC
```

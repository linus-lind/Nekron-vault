---
type: dashboard
tags: [dashboard]
---

# Ideas Pipeline

## Idea → Testing → Validated → Rejected

```dataview
TABLE conviction, status, created
FROM "02_Research/Ideas"
WHERE status = "idea"
SORT conviction DESC
```

## In testing
```dataview
TABLE conviction, created
FROM "02_Research/Ideas"
WHERE status = "testing"
```

## Validated
```dataview
LIST FROM "02_Research/Ideas" WHERE status = "validated"
```

## Rejected (do not re-test blindly)
```dataview
TABLE created FROM "02_Research/Ideas" WHERE status = "rejected"
```

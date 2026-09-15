---
type: dashboard
tags: [dashboard]
---

# Active Experiments

## Running now
```dataview
TABLE model, dataset, hypothesis, created
FROM "05_Experiments"
WHERE status = "running"
SORT created DESC
```

## Recently completed
```dataview
TABLE model, sharpe, max_drawdown, created
FROM "05_Experiments"
WHERE status = "completed"
SORT created DESC
LIMIT 10
```

## Failed / abandoned (learn from these)
```dataview
TABLE model, hypothesis, created
FROM "05_Experiments"
WHERE status = "failed" OR status = "abandoned"
SORT created DESC
```

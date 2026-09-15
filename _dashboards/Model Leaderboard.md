---
type: dashboard
tags: [dashboard]
---

# Model Leaderboard

## Production & staging models
```dataview
TABLE architecture, stage, version, created
FROM "04_Models"
WHERE stage = "production" OR stage = "staging"
SORT stage ASC
```

## All models
```dataview
TABLE architecture, stage, version
FROM "04_Models"
SORT created DESC
```

## Best experiments by Sharpe
```dataview
TABLE model, sharpe, max_drawdown, status
FROM "05_Experiments"
WHERE sharpe != null
SORT sharpe DESC
LIMIT 15
```

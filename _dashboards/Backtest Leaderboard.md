---
type: dashboard
tags: [dashboard]
---

# Backtest Leaderboard

## Ranked by Sharpe (valid only)
```dataview
TABLE model, sharpe, sortino, max_drawdown, cagr, period
FROM "06_Backtests"
WHERE status = "completed" AND sharpe != null
SORT sharpe DESC
```

## Low drawdown (< 20%)
```dataview
TABLE model, sharpe, max_drawdown, period
FROM "06_Backtests"
WHERE max_drawdown != null AND max_drawdown > -0.20
SORT sharpe DESC
```

---
type: model
stage: research       # research | staging | production | retired
architecture: "<% tp.system.prompt('Architecture (e.g. LSTM, XGBoost, Transformer)') %>"
version: "<% tp.system.prompt('Version (e.g. v3)') %>"
owner: me
created: <% tp.date.now("YYYY-MM-DD") %>
tags: [model]
---

# <% tp.file.title %>

**Architecture:** `= this.architecture` · **Stage:** `= this.stage`

## Purpose
What signal/edge is this model meant to capture?

## Inputs / Features
Links to [[Feature]] notes it consumes.

## Experiments using this model
```dataview
TABLE status, sharpe, max_drawdown, created
FROM "05_Experiments"
WHERE contains(model, this.file.name)
SORT created DESC
```

## Backtests using this model
```dataview
TABLE sharpe, max_drawdown, cagr, period
FROM "06_Backtests"
WHERE contains(model, this.file.name)
SORT sharpe DESC
```

## Notes
Known failure modes, retraining cadence, gotchas.

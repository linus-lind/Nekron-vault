---
type: backtest
status: completed      # completed | running | invalid
model: "[[<% tp.system.prompt('Model (e.g. lstm-v3)') %>]]"
experiment: "[[<% tp.system.prompt('Source experiment note') %>]]"
universe: "<% tp.system.prompt('Universe (e.g. sp500)') %>"
period: "<% tp.system.prompt('Period (e.g. 2015-2024)') %>"
# --- headline metrics ---
sharpe:
sortino:
max_drawdown:
cagr:
turnover:
win_rate:
# --- reproducibility ---
git_commit: "<% tp.system.prompt('Code commit SHA') %>"
config_path: "configs/"
mlflow_run_id: ""
created: <% tp.date.now("YYYY-MM-DD") %>
tags: [backtest]
---

# <% tp.file.title %>

Model `= this.model` · Universe `= this.universe` · Period `= this.period`

## Headline
| Sharpe | Sortino | Max DD | CAGR | Turnover | Win rate |
|---|---|---|---|---|---|
| `= this.sharpe` | `= this.sortino` | `= this.max_drawdown` | `= this.cagr` | `= this.turnover` | `= this.win_rate` |

## Reproduce
- Code: https://github.com/linus-lind/Nekron/commit/`= this.git_commit`
- Config: https://github.com/linus-lind/Nekron/blob/`= this.git_commit`/`= this.config_path`
- MLflow: `= this.mlflow_run_id`

## Equity curve
![[ ]]

## Robustness checks
- [x] Out-of-sample / walk-forward holds up
- [ ] No look-ahead / survivorship bias (see [[Frontmatter Schema]])
- [ ] Transaction costs & slippage modeled
- [ ] Stable across regimes

## Verdict
Ship / iterate / reject — and why.

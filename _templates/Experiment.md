---
type: experiment
status: running        # running | completed | failed | abandoned
model: "[[<% tp.system.prompt('Model (e.g. lstm-v3)') %>]]"
dataset: "<% tp.system.prompt('Dataset id') %>"
hypothesis: "<% tp.system.prompt('One-line hypothesis') %>"
# --- results (fill on completion) ---
sharpe:
sortino:
max_drawdown:
cagr:
val_loss:
# --- reproducibility pointers ---
git_commit: "<% tp.system.prompt('Code commit SHA') %>"
config_path: "configs/"
mlflow_run_id: "<% tp.system.prompt('MLflow run id (optional)') %>"
created: <% tp.date.now("YYYY-MM-DD") %>
tags: [experiment]
---

# <% tp.file.title %>

**Hypothesis:** `= this.hypothesis`
**Model:** `= this.model` · **Dataset:** `= this.dataset` · **Status:** `= this.status`

## Reproduce
- Code: https://github.com/linus-lind/Nekron/commit/`= this.git_commit`
- Config: https://github.com/linus-lind/Nekron/blob/`= this.git_commit`/`= this.config_path`
- MLflow run: `= this.mlflow_run_id`

## Setup
What am I changing vs. the last run? What do I expect?

## Results
Paste key metrics here; embed the equity/loss curve.
![[ ]]

## Interpretation
Did the hypothesis hold? Overfit? Regime-specific? What surprised me?

## Next
- [ ] Follow-up experiment: 
- [ ] Promote to backtest? → [[_templates/Backtest]]

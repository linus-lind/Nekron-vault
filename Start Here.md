---
type: home
title: Start Here
tags: [meta]
---

# Nekron — Command Center

The knowledge & decision layer for Nekron, my ML-driven investing app.
Code repo: `github.com/linus-lind/Nekron` · Experiment tracker: **MLflow**

> [!tip] The one habit that makes this vault work
> Every note gets **frontmatter** (the `---` block up top). Consistent properties = live dashboards. See [[Frontmatter Schema]].

## Create (use Templater / QuickAdd hotkey)
- New Experiment → `_templates/Experiment`
- New Backtest → `_templates/Backtest`
- New Idea → `_templates/Idea`
- New Data Source → `_templates/Data Source`
- New Decision (ADR) → `_templates/ADR`
- New Incident → `_templates/Incident`
- New Tuning Campaign → `_templates/HPO Campaign`
- New Tuning Trial → `_templates/HPO Trial`
- Daily Note → `_templates/Daily Note`

## Dashboards
- [[Model Leaderboard]]
- [[Tuning]]
- [[Backtest Leaderboard]]
- [[Active Experiments]]
- [[Ideas Pipeline]]
- [[Live Health]]
- [[Open Tasks]]
- [[Research Reading]]

## Areas
| Area | Folder | Purpose |
|---|---|---|
| Research | `02_Research/` | Papers, concepts, hypotheses |
| Data | `03_Data/` | Source catalog, feature docs |
| Models | `04_Models/` | Model registry |
| Experiments | `05_Experiments/` | Training runs; `HPO/` holds tuning campaigns |
| Backtests | `06_Backtests/` | Validation runs |
| Live | `07_Live/` | Runbooks, incidents, journal |
| Decisions | `08_Decisions/` | ADRs |
| Ops | `09_Ops/` | Infra, RunPod, cost |

## Setup checklist
- [x] Install plugins: Dataview, Templater, QuickAdd, Obsidian Git
- [x] Templater: set template folder to `_templates`
- [ ] QuickAdd: bind hotkeys to the templates above
- [x] GitHub username set (`linus-lind`)
- [ ] Read [[Vault Conventions]] and [[Frontmatter Schema]]
- [ ] `git init` + push vault to GitHub

## Recently touched
```dataview
TABLE type, status, file.mtime AS "Modified"
FROM "" 
WHERE type != "home"
SORT file.mtime DESC
LIMIT 10
```

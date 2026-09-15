---
type: reference
tags: [meta, schema]
---

# Frontmatter Schema

The dashboards only work if property **names and values are consistent**. Decide once, here. When in doubt, copy from a template — don't improvise field names.

## Universal
| Field | Values | Notes |
|---|---|---|
| `type` | experiment · backtest · model · idea · data-source · feature · adr · incident · paper · concept · runbook · daily · dashboard · hpo-campaign · hpo-trial | drives every query |
| `status` | see per-type below | keep the exact strings |
| `tags` | list | broad grouping |
| `created` | `YYYY-MM-DD` | ISO only |

## Status vocabularies (use these exact strings)
- **experiment:** `running` · `completed` · `failed` · `abandoned`
- **backtest:** `running` · `completed` · `invalid`
- **model.stage:** `research` · `staging` · `production` · `retired`
- **idea:** `idea` · `testing` · `validated` · `rejected`
- **data-source:** `active` · `evaluating` · `dropped`
- **feature:** `active` · `experimental` · `deprecated`
- **adr:** `proposed` · `accepted` · `superseded`
- **incident.status:** `open` · `mitigated` · `resolved` ; **severity:** `sev1` · `sev2` · `sev3`
- **paper:** `to-read` · `reading` · `read`
- **hpo-campaign:** `planned` · `running` · `completed` · `abandoned`
- **hpo-trial.status:** `planned` · `running` · `complete` · `failed`
- **hpo-trial.decision:** `baseline` · `accepted` · `rejected` · `tie` · `invalid`
- **hpo-trial.role:** `noise` · `screen` · `confirm` · `replicate` · `sealed`

## Data sources
| Field | Format | Notes |
|---|---|---|
| `grain` | string | e.g. `daily, (date, entity)` |
| `coverage_start`, `coverage_end` | `YYYY-MM-DD` | what the source actually holds, not what a model reads |

An experiment or campaign links to the source note and states its own window separately; a model never reads "everything the file happens to hold".

## Metric conventions (numbers, not strings)
| Field | Format | Example |
|---|---|---|
| `sharpe`, `sortino` | float | `1.42` |
| `max_drawdown` | negative float (fraction) | `-0.18` = -18% |
| `cagr` | float fraction | `0.24` = 24% |
| `turnover`, `win_rate` | float fraction | `0.55` |

> Store metrics as raw numbers (no `%`, no quotes) so Dataview can sort/filter them.

## Reproducibility fields (experiments & backtests)
| Field | Meaning                                                  |
| --------------- | -------------------------------------------------------- |
| `git_commit` | SHA in `linus-lind/Nekron` — the code that produced this |
| `config_path` | path to config within that commit                        |
| `mlflow_run_id` | MLflow run — the metrics source of truth                 |

**Rule:** no experiment or backtest note without a `git_commit`. If you can't point to the exact code, the result isn't reproducible.

## Hyperparameter tuning

Two types, both living in `05_Experiments/HPO/`. The method they follow is [[HPO Protocol]]; the field lists are in its section 9.

| Type | One per | Holds |
|---|---|---|
| `hpo-campaign` | search | the pre-registration, the noise floor, the decision rule, and Dataview ledgers over its trials |
| `hpo-trial` | trial | one challenger's numbers, gates and decision |

Dataset fields on `hpo-campaign`, which every experiment-like note should follow:

| Field | Format | Meaning |
|---|---|---|
| `dataset` | `"[[source-note]]"` | link to the `data-source` note, never a free-text name |
| `data_start`, `data_end` | `YYYY-MM-DD` | the window actually ingested, stated explicitly and never left unbounded |
| `universe` | string | the point-in-time filter, e.g. `top 1000 by daily market cap` |
| `holdout_start`, `holdout_end` | `YYYY-MM-DD` | the range deliberately not ingested |
| `data_key` | hex string | digest of the terminal data-pipeline stage; the machine-checkable form of all of the above |

Three rules specific to these:

- **Specify data by range and source, not by fold count.** A fold count is a consequence of window sizes applied to a date range. It changes when either moves and describes neither.
- **Never name a field `from`.** It is a Dataview query keyword and breaks every query that touches it. Use `param_from` / `param_to`.
- **Most trial fields are pasted, not typed.** `scripts/hpo_stats.py` prints them in frontmatter form; a trial note's hand-written part is the parameter that moved and the reason for the verdict.

## Links, not folders
Prefer `[[wikilinks]]` between related notes (model ↔ experiment ↔ backtest ↔ idea). Folders are coarse buckets; the graph is the real structure.

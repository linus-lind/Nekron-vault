---
type: dashboard
tags: [dashboard, hpo]
---

# Tuning

Hyperparameter campaigns and their trials. Method: [[HPO Protocol]].

## Campaigns
```dataview
TABLE WITHOUT ID
  file.link AS "campaign", model AS "model", status AS "status",
  dataset AS "dataset",
  (string(data_start) + " to " + string(data_end)) AS "window",
  metric AS "metric", sigma_seed AS "σ",
  (string(trials_spent) + " / " + string(trial_budget)) AS "trials"
FROM "05_Experiments/HPO"
WHERE type = "hpo-campaign"
SORT created DESC
```

## Running now
```dataview
TABLE WITHOUT ID
  file.link AS "trial", campaign AS "campaign", role AS "role",
  param AS "parameter", param_to AS "to"
FROM "05_Experiments/HPO"
WHERE type = "hpo-trial" AND status = "running"
SORT trial ASC
```

## Awaiting a decision
```dataview
TABLE WITHOUT ID
  file.link AS "trial", campaign AS "campaign", param AS "parameter",
  delta_over_sigma AS "Δ/σ", t_corrected AS "t*"
FROM "05_Experiments/HPO"
WHERE type = "hpo-trial" AND status = "complete" AND !decision
SORT trial ASC
```

## Invalid — compute spent, no evidence
```dataview
TABLE WITHOUT ID
  file.link AS "trial", campaign AS "campaign", param AS "parameter",
  folds_failed AS "failed", folds_hit_cap AS "capped", folds_diverged AS "diverged",
  runtime_min AS "min"
FROM "05_Experiments/HPO"
WHERE type = "hpo-trial" AND decision = "invalid"
SORT trial ASC
```

## Accepted changes, all campaigns
```dataview
TABLE WITHOUT ID
  file.link AS "trial", campaign AS "campaign", param AS "parameter",
  param_from AS "from", param_to AS "to", t_corrected AS "t*"
FROM "05_Experiments/HPO"
WHERE type = "hpo-trial" AND decision = "accepted"
SORT campaign ASC, trial ASC
```

## Rejected — do not re-test blind
```dataview
TABLE WITHOUT ID
  campaign AS "campaign", param AS "parameter", param_to AS "tried",
  delta_over_sigma AS "Δ/σ"
FROM "05_Experiments/HPO"
WHERE type = "hpo-trial" AND decision = "rejected"
SORT campaign ASC, param ASC
```

## Compute spent per campaign
```dataview
TABLE WITHOUT ID
  key AS "campaign", length(rows) AS "trials",
  round(sum(rows.runtime_min) / 60, 1) AS "hours"
FROM "05_Experiments/HPO"
WHERE type = "hpo-trial"
GROUP BY campaign
```

## Data windows in use
What each campaign reads, and what it holds back. A campaign whose window overlaps another's holdout is a mistake worth seeing.

```dataview
TABLE WITHOUT ID
  file.link AS "campaign", dataset AS "dataset",
  data_start AS "from", data_end AS "to", universe AS "universe",
  (string(holdout_start) + " to " + string(holdout_end)) AS "held back",
  data_key AS "data_key"
FROM "05_Experiments/HPO"
WHERE type = "hpo-campaign"
SORT data_start ASC
```

## Audit exceptions
Trials whose provenance does not satisfy the reproducibility contract in [[Vault Conventions]].

```dataview
TABLE WITHOUT ID
  file.link AS "trial", campaign AS "campaign",
  git_commit AS "commit", git_dirty AS "dirty", data_key AS "data"
FROM "05_Experiments/HPO"
WHERE type = "hpo-trial" AND (git_dirty = true OR git_dirty = "true" OR !git_commit)
SORT trial ASC
```

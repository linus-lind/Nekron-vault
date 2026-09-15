---
type: hpo-campaign
status: planned        # planned | running | completed | abandoned
model: "[[<% tp.system.prompt('Model note (e.g. cae-v1)') %>]]"
protocol_version: 1
# --- what decides (pre-registered, never edited) ---
metric: "<% tp.system.prompt('Selection metric column (e.g. val_total_r2)') %>"
t_hurdle: 3.0
delta_practical:
sigma_seed:
trial_budget: 30
trials_spent: 0
# --- the data (campaign identity) ---
dataset: "[[<% tp.system.prompt('Data source note (e.g. crsp-daily)') %>]]"
data_start: <% tp.system.prompt('Data start (YYYY-MM-DD)') %>
data_end: <% tp.system.prompt('Data end (YYYY-MM-DD)') %>
universe: "<% tp.system.prompt('Universe filter') %>"
holdout_start:         # the range held back from this campaign entirely
holdout_end:
data_key: ""           # set from the first run
# --- reproducibility ---
git_commit: "<% tp.system.prompt('Code commit SHA (must be pushed)') %>"
config_path: "configs/"
created: <% tp.date.now("YYYY-MM-DD") %>
tags: [hpo]
---

# <% tp.file.title %>
Method: [[HPO Protocol]]. 

## Reproduce
- Code: https://github.com/linus-lind/Nekron/commit/`= this.git_commit`
- Config: https://github.com/linus-lind/Nekron/blob/`= this.git_commit`/`= this.config_path`
- Data: `= this.dataset` · `= this.data_start` to `= this.data_end` · `= this.universe` · `data_key = `= this.data_key``
- Held back: `= this.holdout_start` to `= this.holdout_end`
- Entry point: `python -m nekron`

---

## 1. Pre-registration

**Selection set**
Text here.

**Frozen constants**

| Setting | Value | Why frozen |
|---|---|---|
| | | |

**Dataset**

|            |     |
| ---------- | --- |
| window     |     |
| universe   |     |
| grain      |     |
| target     |     |
| `data_key` |     |
| held back  |     |

**Fold schedule**

|                | periods |
| -------------- | ------- |
| train          |         |
| val            |         |
| test           |         |
| purge          |         |
| step           |         |
| `max_folds`    |         |
| folds produced |         |

---

## 2. Noise floor

Measured in [[<% tp.file.title %>-T00-noise-floor]].

| | value |
|---|---|
| seeds | |
| pooled mean | |
| `sigma_seed` | |
| `range_band` | |
| `screen_band_1_seed` | |
| `confirm_band_3_seeds` | |

**A/A control:** two baseline replicates pushed through the full decision rule.
- [ ] the control does **not** produce an acceptance

---

## 3. Metrics
### Decides

| Metric | Role | Rule |
|---|---|---|
| | primary | the only metric that can accept a trial |
| | guardrail | can only reject, never accept |
### Informs

Never enters a decision. Recorded so a result can be explained afterwards.

| Metric | Read for |
|---|---|
| | |

---

## 4. Hard gates

Admissibility only. A failed gate makes a trial `invalid`, not `rejected` — the run is not evidence either way and the coordinate stays open. Whether a change is *good* is §5; the guardrail lives there, not here.

**Machine-checked** — `scripts/hpo_stats.py` writes these into the trial's frontmatter:

| Gate                      | Condition                                                   |
| ------------------------- | ----------------------------------------------------------- |
| every fold ran            | `folds_failed == 0`                                         |
| no fold diverged          | `folds_diverged == 0`                                       |
| no fold ran out of budget | `folds_hit_cap == 0`                                        |
| same data                 | `data_key` equals the campaign's                            |
| same code, clean tree     | `git_commit` equals the campaign's and `git_dirty == false` |
| same schedule             | `folds_planned` equals the campaign's fold count            |
| a real change             | `config_hash` differs from the incumbent's                  |

**Per-fold** — each metric at the selected epoch, and the condition must hold on *every* fold. Values pinned from T00 are frozen once the noise run reports the baseline's.

| Gate | Condition |
|---|---|
| something was learned | `fold/best_epoch >= 1` |
| | |

**Read by hand:**

- [ ] exactly one coordinate, or one declared coupled pair, moved

A capped run is re-run under a raised campaign-wide epoch budget, never discarded: dropping capped runs biases the search toward whatever converges fastest.

---

## 5. Decision rule

Verbatim from [[HPO Protocol]] §5, with this campaign's numbers:

```
accept  iff  t_corrected >= t_hurdle
        and  delta_pooled >= delta_practical (band for the seed count used)
        and  no guardrail regressed beyond its own band
        and  every hard gate passed
tie     iff  abs(delta_pooled) <= delta_practical   -> parsimony ladder
invalid iff  any hard gate failed
reject  otherwise
```

**Two stages.** 
Screen at 1 seed against `screen_band_1_seed`; confirm at 3 common seeds against `confirm_band_3_seeds` and the `t` hurdle. On adoption, the incumbent's stored value is its confirmation value, not the value that won the trial.

**Parsimony ladder.**
(ties): keep incumbent > fewer parameters > simpler architecture > less regularization machinery > shorter wall clock > keep incumbent.

---

## 6. Parameter schedule

Ordered by expected leverage; two full passes.

| # | Coordinate | Values | Coupled with | Pass 1 | Pass 2 |
|---|---|---|---|---|---|
| 1 | | | | | |

**Coupled pairs** — moved as a joint grid, never as two 1-D sweeps. Coordinate descent moves along one axis at a time, so on a diagonal ridge every single-axis step goes downhill in both directions and the search stops at a point optimal in each coordinate separately and in neither jointly.

| Pair | Why they move together |
|---|---|
| | |

**Stopping rule:** a full pass accepts nothing, or the trial budget is spent.

---

## 7. Trial ledger

```dataview
TABLE WITHOUT ID
  link(file.path, "T" + padleft(string(trial), 2, "0")) AS "T",
  role AS "role", param AS "parameter",
  param_from AS "from", param_to AS "to",
  seeds_candidate AS "n",
  delta_pooled AS "Δ", delta_over_sigma AS "Δ/σ", t_corrected AS "t*",
  fold_win_rate AS "win", decision AS "decision"
FROM "05_Experiments/HPO"
WHERE type = "hpo-trial" AND campaign = this.file.link
SORT trial ASC
```

### Accepted — the incumbent path
```dataview
TABLE WITHOUT ID
  link(file.path, "T" + padleft(string(trial), 2, "0")) AS "T",
  param AS "parameter", param_from AS "from", param_to AS "to",
  candidate_pooled AS "new value", t_corrected AS "t*"
FROM "05_Experiments/HPO"
WHERE type = "hpo-trial" AND campaign = this.file.link AND decision = "accepted"
SORT trial ASC
```

### Rejected — do not re-test blind
```dataview
TABLE WITHOUT ID
  link(file.path, "T" + padleft(string(trial), 2, "0")) AS "T",
  param AS "parameter", param_to AS "tried", delta_over_sigma AS "Δ/σ"
FROM "05_Experiments/HPO"
WHERE type = "hpo-trial" AND campaign = this.file.link AND decision = "rejected"
SORT trial ASC
```

### Invalid — needs re-running
```dataview
TABLE WITHOUT ID
  link(file.path, "T" + padleft(string(trial), 2, "0")) AS "T",
  param AS "parameter",
  folds_failed AS "failed", folds_hit_cap AS "capped", folds_diverged AS "diverged"
FROM "05_Experiments/HPO"
WHERE type = "hpo-trial" AND campaign = this.file.link AND decision = "invalid"
SORT trial ASC
```

### Cost
```dataview
TABLE WITHOUT ID
  sum(rows.runtime_min) AS "minutes", length(rows) AS "trials"
FROM "05_Experiments/HPO"
WHERE type = "hpo-trial" AND campaign = this.file.link
GROUP BY true
```

---

## 8. Final configuration

Filled when the campaign stops — a full pass accepts nothing, or the trial budget is spent.

| | value |
|---|---|
| champion `config_hash` | |
| overrides vs baseline | |
| pooled selection metric — champion | |
| pooled selection metric — original baseline | |
| screens spent (N) | |
| expected best-of-N gap under the null (`σ·sqrt(2 ln N)`) | |
| champion gain, less that premium | |
| total compute | |

> [!warning] What this number is, and is not
> The pooled figure above is the maximum of N selected draws, not an unbiased estimate. A campaign cannot price its own selection premium — only an untouched holdout can. Quote the champion as "best configuration found", never as an out-of-sample result.

- [ ] champion config committed and the SHA pushed
- [ ] every trial note has a decision
- [ ] campaign `status` set to `completed`
- [ ] head-to-head champion against the original baseline at 5 common seeds, recorded as the last trial
- [ ] the holdout is still untouched
- [ ] promoted to an [[Experiment]] / [[Backtest]] note, or [[ADR]] recorded

**Verdict:**

---

## 9. Protocol changelog

Amendments apply forward only. Never retro-apply a threshold.

| Date | v | Change | Why |
|---|---|---|---|
| <% tp.date.now("YYYY-MM-DD") %> | 1 | Pre-registered. | |

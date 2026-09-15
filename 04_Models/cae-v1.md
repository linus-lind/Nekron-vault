---
type: model
stage: research       # research | staging | production | retired
architecture: "Conditional autoencoder"
version: "v1"
owner: me
created: 2026-09-09
tags: [model, asset-pricing]
---

# cae-v1

**Architecture:** `= this.architecture` · **Stage:** `= this.stage`

Code: `src/nekron/asset_pricing/conditional_autoencoder/` · Config: `configs/conditional_autoencoder.yaml` · MLflow experiment: `conditional_autoencoder`

## Purpose
A latent-factor asset pricing model in the Gu-Kelly-Xiu form. Two networks share an output width `K`: a **beta network** maps each stock's characteristics at date `t` to its factor loadings, and a **factor network** maps that period's managed-portfolio returns to the `K` factors. A stock's return is priced as the inner product of the two, `r = beta . f` — with **no per-stock alpha**, which is the whole claim: whatever a characteristic explains, it explains through a loading on a common factor.

## Inputs
- Panel: [[crsp-daily]], top 1000 by market capitalization on each date, via `configs/data/crsp.yaml`.
- Beta inputs: every feature column except the target and anything explicitly excluded, rank-transformed to `[-1, 1]` within each cross-section.
- Factor inputs: 49 named characteristics forming the managed portfolios.
- Target: `fwd_ret_1d`, cross-sectionally z-scored per period.

## How it is scored
Two R-squared statistics, both uncentered — the benchmark is zero, not the cross-sectional mean:

- **Total R-squared** — fit using the *contemporaneous* factor. What the model explains with hindsight about the factor realization.
- **Predictive R-squared** — the same errors against `beta . lambda_{t-1}`, where `lambda` is a forecast built only from earlier periods. The out-of-sample statistic.

Pooled as a ratio of summed errors to summed totals over every scored period, never as a mean of per-fold ratios.

There is no Sharpe ratio, no factor-portfolio return series and no alpha test in the package. If the campaign's verdict is meant to survive into a backtest, those have to be built first.

## Known failure modes
- **Factor collapse.** `factors/effective` — a participation ratio over the factors' contribution shares — falls toward 1 when only one factor is doing work. `factors/max_abs_corr` near 1 means two factors are duplicates. Both are per-epoch metrics and both are campaign gates.
- **A non-finite factor is permanent.** The predictive forecast carries state forward across the whole fold, so one `inf` entering it never decays out.
- **Rotation indeterminacy.** Factors are not comparable across folds; only the R-squared statistics are. Nothing that averages factor series across folds is meaningful.
- **Device-dependent numbers.** `train.device: auto` resolves to CUDA, then MPS, then CPU. The resolved device is recorded as a run tag; comparing runs across devices compares kernels as much as configurations.

## Tuning
```dataview
TABLE WITHOUT ID
  file.link AS "campaign", status AS "status", metric AS "metric",
  sigma_seed AS "σ", (string(trials_spent) + " / " + string(trial_budget)) AS "trials"
FROM "05_Experiments/HPO"
WHERE type = "hpo-campaign" AND contains(model, this.file.name)
SORT created DESC
```

## Experiments using this model
```dataview
TABLE status, sharpe, max_drawdown, created
FROM "05_Experiments"
WHERE type = "experiment" AND contains(model, this.file.name)
SORT created DESC
```

## Backtests using this model
```dataview
TABLE sharpe, max_drawdown, cagr, period
FROM "06_Backtests"
WHERE contains(model, this.file.name)
SORT sharpe DESC
```

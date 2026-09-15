---
type: hpo-trial
campaign: "[[hpo-20260909-cae-v1]]"
trial: 0
role: noise
status: completed
decision: baseline
param: train.seed
param_from: 42
param_to: 42,482,7319,56,2047,12,1238,195
overrides: train.seed=42,482,7319,56,2047,12,1238,195
created: 2026-09-09
tags:
  - hpo
  - conditional-autoencoder
---

# hpo-20260909-cae-v1-T00-noise-floor

`= this.param`: `= this.param_from` -> `= this.param_to` · role `= this.role` · campaign `= this.campaign`

## Hypothesis
Not a hypothesis. Five runs of one configuration differing only in the seed, to measure how far apart replicates land on the pooled held-out R-squared. Everything the campaign accepts or rejects later is a multiple of this number.

## Command
```
python -m nekron.asset_pricing.conditional_autoencoder --multirun \
    train.seed=42,482,7319,56,2047,12,1238,195 \
    +mlflow.tags.campaign=hpo-20260909-cae-v1 \
    +mlflow.tags.trial=T00 \
    +mlflow.tags.role=noise

python scripts/hpo_stats.py noise \
    --experiment conditional_autoencoder --campaign hpo-20260909-cae-v1 --trial T00
```

## Gates
Machine-checked: `folds_failed` `= this.folds_failed` · `folds_hit_cap` `= this.folds_hit_cap` · `folds_diverged` `= this.folds_diverged` · `git_dirty` `= this.git_dirty`

- [ ] all five runs completed every planned fold
- [ ] no run hit the epoch cap — if any did, raise the campaign-wide budget and re-measure
- [ ] no run diverged
- [ ] all five share one `config_hash` and one `data_key`
- [ ] `git_dirty == false` on all five (tracked edits only; `git_untracked` is recorded, not gated)

## Results

Run once per decision metric — the primary sets the acceptance band, the guardrail sets the band it may not fall through. Record both.

| | value |
|---|---|
| pooled per seed | |
| `pooled_mean` | |
| `sigma_seed` | |
| `range_band` | |
| `screen_band_1_seed` | |
| `confirm_band_3_seeds` | |
| `t_quantile` (4 df) | 2.132 |
| median `fit/fit_seconds_total` | |

Sanity checks on the number before it is used as a threshold:

- [ ] `sigma_seed` is small against the pooled level — if replicates of one configuration differ by a large fraction of the score, no configuration comparison in this campaign will be decidable and the fold count or the sample has to grow first
- [ ] `range_band` and `2.13 x sigma_seed` are of the same order — a large gap between them means one replicate is an outlier and the standard deviation is not describing the others
- [ ] `cudnn_benchmark` and `tf32_matmul` are `false` on every run — otherwise part of this spread is hardware nondeterminism rather than initialization, and it will not reproduce
- [ ] a same-seed re-run reproduces its pooled value exactly; if it does not, the nondeterminism floor is above the seed noise and must be measured separately
- [ ] `val_total_r2` is not far above `test_total_r2` on any replicate — a large gap says early stopping is choosing the epoch on noise, and `train.min_delta` or the patience needs revisiting before the campaign starts

## A/A control
Two of the eight replicates pushed through the full decision rule as if they were incumbent and challenger:

```
python scripts/hpo_stats.py compare --incumbent <seed-42 run> --candidate <seed-482 run> --sigma <sigma_seed>
```

- [ ] the control does **not** clear the acceptance rule

If it does, the gate is broken and nothing the campaign accepts afterwards means anything.

## Decision

> [!note] BASELINE
> The incumbent is the configuration as committed. No comparison is made here.

## Action
- [ ] `sigma_seed` and the bands copied into [[hpo-20260909-cae-v1]] §2
- [ ] `delta_practical` set in the campaign frontmatter
- [ ] `data_key` and `git_commit` pinned in the campaign frontmatter
- [ ] `train.min_delta` pinned from the per-epoch validation curves in `history.parquet`
- [ ] `train.epochs` confirmed generous enough that no fold hits the cap
- [ ] T01 opened on `optim.lr`

## Notes

**Superseded setup, 2026-09-15.** The campaign now ingests `1990-01-01 .. 2019-12-31` and selects on the folds' held-out windows; 2020-2025 is held back untouched for the finished strategy. Nothing is sealed inside the campaign and `walk_forward.max_folds` stays `null`. Baseline `config_hash` is `6ab19a60a6b0`, `data_key` `23e9ea00c76a`.

**First attempt, 2026-09-09 14:43 — failed before any run started.** The `--multirun` over the five seeds produced five job directories under `multirun/2026-09-09/14-43-06/`; every log ends at

```
nekron.cv.schedule - 1 of 1508 dates produced no cross-section of at least
data.min_cross_section=100; folds are cut over the 1507 that remain.
```

and no `mlflow.db` was ever created. A fold spans `1260 + 252 + 252 + 2 x 0 = 1764` periods against 1507 available, so `generate_folds` raises `AdapterError`. Nothing was recorded and nothing was lost. The window was widened afterwards and the schedule now works. The seeds above are the ones that attempt used, kept so the re-run is the same experiment.

## Revisions
Append-only. A correction is a dated line here, never a rewritten field above.

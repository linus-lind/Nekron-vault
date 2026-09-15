---
type: reference
tags: [meta, hpo, method]
protocol_version: 1
created: 2026-09-09
---

# HPO Protocol

How a hyperparameter search is run and recorded in this vault, for any model. A campaign note fills in the model-specific blanks; the method lives here so it is written once and cannot quietly differ between campaigns.

Templates: [[HPO Campaign]] · [[HPO Trial]]. Dashboard: [[Tuning]]. Tooling: `scripts/hpo_stats.py` in the code repo.

> [!warning] The one rule that makes the rest work
> Everything in sections 1-6 is **pre-registered**: written down before the first trial and never edited afterwards. Changing a threshold after seeing a result is how a search convinces itself of things that are not true. Amendments go in the campaign's protocol changelog with a new `protocol_version`, and they apply forward only.

---

## 1. Vocabulary

| Term | Meaning |
|---|---|
| **Campaign** | One search over one model on one dataset under one protocol version. Its identity is `{dataset + date range + universe, code SHA, fold schedule, metric definition, protocol version}`. `data_key` is the machine-checkable form of the first three. |
| **Incumbent** | The configuration currently held to be best. Starts as the baseline. |
| **Challenger** | A configuration differing from the incumbent in exactly one coordinate (or one coupled pair). |
| **Trial** | One challenger, evaluated and decided. Numbered `T01`, `T02`, ... `T00` is the noise floor. |
| **Screen** | A cheap evaluation of a challenger at one seed, against a wide band. |
| **Confirm** | A re-evaluation of a promising challenger at several seeds, against the real band, before it may become incumbent. |
| **Generation** | Everything that comes after a change to the campaign's identity. A new generation re-measures the noise floor and starts a new campaign note. |

---

## 2. What a campaign pre-registers

Six things, all before `T01`:

1. **The selection metric** — one number, one direction, one place it is read from.
2. **The selection set** — which window decides, and what is held out of the campaign entirely.
3. **The hard gates** — conditions that make a run's number admissible at all.
4. **The decision rule** — the arithmetic that turns two runs into accept / reject / tie.
5. **The parameter schedule** — which coordinates are searched, in what order, over what values, and which are frozen.
6. **The trial budget** — how many screens the campaign may spend, and when it stops.

A campaign missing any of these is not auditable, because after the fact there is no way to tell a rule from a rationalization.

---

## 3. The selection set

A walk-forward fold has three windows: `train`, `val`, `test`, with purge gaps between them. They sit at two different levels, and conflating them is the usual mistake.

- **`train` and `val` are inner.** The model is fitted on `train`; `val` chooses the epoch to stop at. Both belong to fitting *one* model. A learner that needs no early stopping — a random forest, say — needs no `val` at all.
- **`test` is the fold's held-out score, and it is the outer signal.** It is scored once, with the epoch already chosen, and it is what one configuration is compared with another on. In plain k-fold language this is simply "the fold's score"; the name `test` is a collision with the deep-learning convention, not a different role.
- **The final holdout sits outside the campaign entirely.** Not a block of folds — a date range that is never ingested, reserved for the end-to-end evaluation of the finished strategy. The campaign's data window must exclude it.

That is ordinary nested selection:

| level | chooses | on | how often |
|---|---|---|---|
| inner | the epoch | `val` | every epoch of every fold |
| outer | the configuration | `test`, pooled across folds | once per trial |
| final | nothing — confirms | the external holdout | once, ever |

> [!warning] Do not select on `val`
> `val` is consumed by early stopping, which takes a maximum over epochs of a noisy curve. The expected maximum grows with the noise, so a configuration whose training curve is noisier — higher learning rate, weaker regularization — collects a larger free boost. Ranking configurations by their `val` score therefore ranks them partly by how noisy they are. That is a bias in the *ranking*, not merely in the level, and ranking is the entire job.
>
> `val` remains worth logging and worth reading: a validation score far above the held-out one says the epoch was chosen on noise. It is a diagnostic of the stopping policy, never a selection metric.

> [!note] What the campaign still spends
> Selecting on the folds' held-out windows does consume them — after `N` trials the pooled held-out number is the maximum of `N` draws, not an unbiased estimate. That is the normal cost of model selection and it is why the external holdout exists. See §7.

---

## 4. The noise floor (`T00`)

Run the baseline configuration `n` times changing **only** `train.seed`. The spread of the headline number across those runs is the floor below which no comparison means anything.

```
python -m <model> --multirun train.seed=42,482,7319,56,2047 +mlflow.tags.trial=T00 +mlflow.tags.role=noise
python scripts/hpo_stats.py noise --experiment <name> --config-hash <hash>
```

The five runs share a `config_hash` (the seed is excluded from it by construction), which is what makes them findable as replicates rather than as five unrelated runs.

Two bands come out, and they fail in opposite directions:

| Band | Definition | Assumes |
|---|---|---|
| `range_band` | `max(pooled) - min(pooled)` over the replicates | nothing at all |
| `screen_band_1_seed` | `t_{.95,n-1} · σ̂ · sqrt(1/n + 1/1)` | approximate normality |
| `confirm_band_3_seeds` | `t_{.95,n-1} · σ̂ · sqrt(1/n + 1/3)` | approximate normality |

Three things about these numbers that are easy to get wrong:

- **`σ̂` is the spread of a single run; a comparison is a difference of two.** The standard error of a difference of means is `σ · sqrt(1/n_base + 1/n_cand)`, which is why a one-seed challenger faces a *wider* band than a three-seed one. "Beat the baseline by 2σ" is not a rule, it is a category error.
- **Five seeds barely pin down σ.** With four degrees of freedom the 95% interval for σ runs from `0.60·σ̂` to `2.87·σ̂`. The Student-t quantile (2.13 at 4 df, against 1.64 for a normal) is the price of that, and it is not optional.
- **Seed-only variation is a lower bound on run-to-run variance.** It excludes data resampling, which a fixed walk-forward schedule cannot vary. Treat `σ̂` as a floor on uncertainty, never as a full account of it. If the campaign runs on CUDA with `cudnn_benchmark` or TF32 enabled — both recorded as run tags — part of `σ̂` is hardware nondeterminism rather than initialization.

**A/A control.** Before trusting the gate, push two baseline replicates through the full decision rule as if they were incumbent and challenger. If that "wins", the gate is broken and every acceptance so far is suspect.

---

## 5. The decision rule

### 5.1 Admissibility first

A trial that fails a **hard gate** is `invalid`, not `rejected`. The distinction matters: rejected means the change did not help, invalid means the run does not constitute evidence either way, and the coordinate is still open.

Gate catalog — a campaign selects from these and pins the thresholds:

| Gate | Read from | Why it disqualifies |
|---|---|---|
| every planned fold completed | `folds/failed == 0` | a partial sweep is scored on a different sample from its comparator |
| no fold diverged | `folds/diverged == 0` | non-finite metrics are dropped, so divergence looks like a merely bad run |
| no fold hit the epoch cap | `folds/hit_epoch_cap == 0` | a capped run measures the budget, not the configuration |
| best epoch not degenerate | `fold/best_epoch >= 1` per fold | selection at epoch 0 means nothing was learned |
| identical data | `data_key` equals the campaign's | a re-pulled source file changes the problem, not the answer |
| identical code | `git_commit` equals the campaign's, `git_dirty == false` | a dirty tree makes the SHA a false claim |
| identical schedule | `folds/planned` and the `cv_*` tags match | a moved window is a different question |
| one coordinate moved | `config_hash` differs, and the override string names one parameter | two changes at once cannot be attributed |

> [!tip] A capped run is re-run, not discarded
> Discarding runs that hit the epoch cap silently biases the search toward whatever converges fastest — low learning rates, heavy penalties and wide networks all need more epochs. The correct response is to raise `train.epochs` for the **whole campaign**, note it as a protocol amendment, and re-run the incumbent under the new budget so the comparison stays paired.

### 5.2 The comparison

Let `m[k,s]` be the per-fold selection metric for fold `k` under seed `s`. Both sides use the **same seeds** — common random numbers, which is free variance reduction — and the **same folds**.

```
d[k,s] = m_challenger[k,s] - m_incumbent[k,s]
d_k    = mean over s of d[k,s]                      # seeds averaged inside a fold
d̄      = mean over k of d_k
s_d    = sd over k of d_k                           (ddof = 1)
SE     = s_d · sqrt( 1/K + n_test/n_train )         # Nadeau-Bengio
t*     = d̄ / SE
Δ      = pooled(challenger) - pooled(incumbent)     # ratio of sums, not mean of ratios
```

Seeds are averaged *within* a fold before folds are differenced: three seeds over eleven folds are eleven paired observations, not thirty-three. Claiming thirty-three is the single easiest way to manufacture significance.

`scripts/hpo_stats.py compare` computes all of it and prints it as frontmatter.

> [!abstract] Why the standard error is inflated
> Walk-forward folds are not independent. Adjacent training windows overlap — entirely, under an expanding window — and returns are regime-persistent, so `s_d/sqrt(K)` is far too small and a t-statistic built on it is far too large. The Nadeau-Bengio term `n_test/n_train` prices the reused training data: with a 1260-period train window and a 252-period test window it more than doubles the naive error.

### 5.3 The verdict

| Verdict | Condition |
|---|---|
| `invalid` | any hard gate failed |
| `accepted` | `t* >= t_hurdle` **and** `Δ >= δ_practical` **and** no guardrail metric regressed by more than its own band |
| `tie` | `abs(Δ) <= δ_practical` — resolved by the parsimony ladder |
| `rejected` | otherwise |

`t_hurdle` defaults to **3.0**. It is not arbitrary: it is approximately the Bonferroni-corrected one-sided 5% quantile for a campaign of ~30 trials at ~15 folds, and it coincides with the `t > 3.0` hurdle the finance literature applies to claimed discoveries. One number, no per-trial multiplicity bookkeeping.

`δ_practical` is the smallest improvement worth having, fixed before the search. It must be at least the applicable band from §4 — `screen_band_1_seed` for a screen, `confirm_band_3_seeds` for a confirmation.

**Two stages.** A challenger is screened at one seed against the wide band. Only if it clears that is it confirmed at three common seeds against the narrower band and the `t*` hurdle. Screening rejects most changes for one run apiece; confirmation is what stops a configuration being adopted on a lucky draw.

**On adoption, re-baseline.** The value stored for the new incumbent is its *confirmation* value at fresh seeds, never the value that won it the trial. Every accepted increment is conditioned on having been large, so it is upward-biased; carrying those values forward makes the champion's number the sum of a chain of biases.

### 5.4 The parsimony ladder

Ties are broken mechanically, in this order, so the tie-break is not a judgment call:

1. Keep the incumbent value. No change is the default.
2. Fewer trainable parameters.
3. Simpler architecture — fewer layers before narrower ones.
4. Less regularization machinery — a disabled mechanism beats a tuned one.
5. Shorter wall clock.
6. Closer to the reference implementation's value.

---

## 6. Search strategy

Greedy coordinate descent — one coordinate at a time, adopting improvements as they come — is a budget heuristic, not a search. It is the right choice under limited compute, with these safeguards:

| Failure mode | Safeguard |
|---|---|
| interactions between coordinates | move known-coupled pairs as a small joint grid, never as two 1-D sweeps; the campaign lists its coupled pairs |
| path dependence on the order | fix the order by prior importance and pre-register it; run **two full passes** |
| stale earlier coordinates | a coordinate accepted in pass 1 is re-tested in pass 2 against the moved incumbent |
| winner's curse | screen-then-confirm, plus re-baselining on adoption (§5.3) |
| a non-separable surface | if more than three or four coordinates matter jointly, spend part of the budget on a random joint search first and coordinate-refine the best point |

**Stopping.** The campaign ends when a full pass accepts nothing, or when the trial budget is spent — whichever comes first, declared in advance. "Stop when it looks good" is optional stopping and invalidates the multiplicity arithmetic.

---

## 7. Multiplicity, and what the final number is worth

With `N` screens against one selection set, the best of them is expected to beat the baseline by roughly `σ·sqrt(2 ln N)` **even if nothing works**:

| N | 5 | 10 | 20 | 30 | 100 |
|---|---|---|---|---|---|
| expected best-of-N gap | 1.16σ | 1.54σ | 1.87σ | 2.04σ | 2.51σ |

Which is exactly why a naive "beat the noise by 2σ" rule is calibrated to the null's expected winner. Three practical consequences:

1. **Declare `N` in advance.** Without it no correction can be computed. Exceeding it raises the hurdle rather than passing quietly.
2. **The external holdout is the claim.** The campaign's own pooled number is a selected maximum; the holdout is the only place it can be priced. The gap between the two is the campaign's measurement of how much it overfit its selection set. Record it when the holdout is finally opened.
3. **Log everything, including what failed.** A record that keeps only winners is not auditable, and the rejected trials are what stop a coordinate being re-tested blind.

---

## 8. Audit and versioning

Every trial note carries, without exception:

| Field | Source |
|---|---|
| `git_commit`, `git_dirty` | run tags, set by `nekron.provenance` on every run |
| `config_hash` | run param — the configuration's identity, seed excluded |
| `data_key` | run param — the digest of the terminal data-pipeline stage |
| `device`, torch/CUDA versions, determinism flags | run tags |
| `runs` | MLflow parent run ids, one per seed |
| `overrides` | the exact command-line override string |
| `seeds` | the seeds used, listed |
| gate counts | `folds/*` run metrics |
| `runtime_min` | `fit/fit_seconds_total` |

Rules:

- **Append-only.** A trial note is not edited after its decision. A correction is a dated line under `## Revisions`, never a rewritten field.
- **The commit must be pushed.** Per [[Vault Conventions]] a `git_commit` has to resolve on GitHub. A campaign pinned to an unpushed SHA is not reproducible by anyone, including its author later.
- **Identity changes end the campaign.** If `data_key`, the fold schedule, the metric definition, the protocol version or any code affecting the metric moves, the campaign is over. Start a new generation, re-measure the noise floor, and link back.
- **Cache fingerprinting.** Under `cache.fingerprint: stat` a source file replaced in place can produce a stale cache hit that the seed cannot see and `data_key` will not move. A campaign that will run for weeks should use `content`.

---

## 9. Frontmatter fields

Registered in [[Frontmatter Schema]]. Fields marked *auto* are printed by `scripts/hpo_stats.py` and pasted verbatim.

**`hpo-campaign`** — `status` (`planned` · `running` · `completed` · `abandoned`), `model`, `protocol_version`, `metric`, `t_hurdle`, `delta_practical`, `sigma_seed`, `trial_budget`, `trials_spent`, `dataset`, `data_start`, `data_end`, `universe`, `holdout_start`, `holdout_end`, `data_key`, `git_commit`, `config_path`, `created`.

The data is specified as **a source note plus a date range plus a universe**, not as a count of folds. A fold count is a consequence of the window sizes applied to that range: it changes when either moves, describes neither on its own, and cannot be compared between campaigns. It belongs in the campaign's schedule table as a derived quantity.

**`hpo-trial`** — `campaign`, `trial` (int), `role` (`noise` · `screen` · `confirm` · `replicate` · `sealed`), `status` (`planned` · `running` · `complete` · `failed`), `decision` (`baseline` · `accepted` · `rejected` · `tie` · `invalid`), `param`, `param_from`, `param_to`, `overrides`, then *auto*: `metric`, `seeds_incumbent`, `seeds_candidate`, `folds_compared`, `incumbent_pooled`, `candidate_pooled`, `delta_pooled`, `delta_per_fold_mean`, `delta_sd_across_folds`, `se_naive`, `se_corrected`, `window_ratio`, `t_corrected`, `fold_win_rate`, `fold_wins`, `sigma_seed`, `delta_over_sigma`, `incumbent_runs`, `folds_planned`, `folds_completed`, `folds_failed`, `folds_hit_cap`, `folds_diverged`, `runtime_min`, `config_hash`, `data_key`, `git_commit`, `git_dirty`, `device`, `seeds`, `runs`.

> [!bug] Do not name a field `from`
> `from` is a Dataview query keyword and a field of that name breaks every query that touches it. Hence `param_from` / `param_to`.

---

## 10. Related

- [[Vault Conventions]] — naming, the GitHub bridge, the reproducibility contract
- [[Frontmatter Schema]] — the registered vocabularies
- [[Tuning]] — every campaign and its open trials

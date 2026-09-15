---
type: hpo-trial
campaign: "[[<% tp.system.prompt('Campaign note') %>]]"
trial: <% tp.system.prompt('Trial number (integer)') %>
role: screen           # noise | screen | confirm | replicate | sealed
status: running        # planned | running | complete | failed
decision:              # baseline | accepted | rejected | tie | invalid
# --- what moved (typed by hand) ---
param: "<% tp.system.prompt('Coordinate, dotted (e.g. optim.lr)') %>"
param_from:
param_to:
overrides: ""
# --- evidence: paste `hpo_stats.py compare` output below this line, unedited ---
created: <% tp.date.now("YYYY-MM-DD") %>
tags: [hpo]
---

# <% tp.file.title %>

`= this.param`: `= this.param_from` -> `= this.param_to` · role `= this.role` · campaign `= this.campaign`

## Hypothesis
One line. What should improve, and why.

## Command
```
python -m <model> <overrides> +mlflow.tags.campaign=<campaign> +mlflow.tags.trial=T<nn> +mlflow.tags.role=<role>
python scripts/hpo_stats.py compare --incumbent <run-id> --candidate <run-id> --sigma <sigma_seed>
```

## Gates
Machine-checked: `folds_failed` `= this.folds_failed` · `folds_hit_cap` `= this.folds_hit_cap` · `folds_diverged` `= this.folds_diverged` · `git_dirty` `= this.git_dirty`

- [ ] exactly one coordinate, or one declared coupled pair, moved
- [ ] `fold/best_epoch >= 1` on every fold
- [ ] every health metric within its pinned range

Any of these failing makes the trial `invalid`, not `rejected`. The guardrail is a
decision input, not a gate — it belongs under Decision.

## Decision

> [!success] ACCEPTED
> Replace with the callout matching the verdict, delete the rest:
> `[!success] ACCEPTED` · `[!failure] REJECTED` · `[!warning] TIE - kept incumbent` · `[!danger] INVALID - gate failed`

One or two sentences. Cite the numbers that decided it, not the ones that did not.

## Action
- [ ] incumbent updated in [[HPO Campaign]] / re-baselined at fresh seeds
- [ ] next coordinate opened

## Notes

## Revisions
Append-only. A correction is a dated line here, never a rewritten field above.

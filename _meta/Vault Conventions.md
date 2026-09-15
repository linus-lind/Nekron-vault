---
type: reference
tags: [meta]
---

# Vault Conventions

## Golden rules
1. **Every note gets frontmatter.** No `type`, no dashboard. See [[Frontmatter Schema]].
2. **Notes hold pointers + interpretation.** Code → git, metrics → MLflow, data → storage. Here lives the *why*.
3. **No secrets, ever.** API keys, broker credentials, `.env` — never in the vault. `.gitignore` guards common patterns, but discipline is the real defense.
4. **Reproducibility contract:** experiments/backtests must carry `git_commit`.
5. **Capture fast, organize light.** Dump into `00_Inbox/`, refine later. Links > folder-shuffling.

## Naming
- Experiments: `exp-YYYYMMDD-<model>-<short>` e.g. `exp-20260818-lstm-v3-vol-target`
- Backtests: `bt-YYYYMMDD-<model>-<universe>`
- Models: `<arch>-v<n>` e.g. `lstm-v3`
- ADRs: `ADR-NNN-<slug>` e.g. `ADR-001-polygon-over-iex`
- Incidents: `INC-YYYYMMDD-<slug>`
- Tuning campaigns: `hpo-YYYYMMDD-<model>` e.g. `hpo-20260909-cae-v1`
- Tuning trials: `<campaign>-T<nn>-<slug>`, inside a folder named after the campaign
  (campaign-qualified because two campaigns would otherwise both own a note called `T01`)

## The GitHub bridge
Code repo: `github.com/linus-lind/Nekron`.
- Reference commits by SHA in `git_commit`; templates build clickable commit/blob URLs.
- GitHub username is `linus-lind`; templates already point to `github.com/linus-lind/Nekron`.

## Hyperparameter tuning
One campaign note plus one note per trial, both in `05_Experiments/HPO/`. The method — noise floor, gates, decision rule, multiplicity, audit — is [[HPO Protocol]]; it is written once and instantiated per model, so campaigns cannot quietly diverge on how they decide things. `scripts/hpo_stats.py` in the code repo prints a trial's numbers as frontmatter, so a trial note is mostly paste.

## Auto-documenting experiments (do this once you're coding)
Have your training/backtest scripts write a note into `05_Experiments/` or `06_Backtests/` on completion — emit the frontmatter (metrics, `git_commit`, `mlflow_run_id`) and a stub body. Then dashboards populate with zero manual effort. Ask Claude to build the note-writer when you get there.

## Plugin setup
| Plugin | Role | Priority |
|---|---|---|
| Dataview | dashboards from frontmatter | essential |
| Templater | dynamic templates | essential |
| QuickAdd | hotkey → new structured note | essential |
| Obsidian Git | auto-commit/backup vault to GitHub | essential |
| Tasks | task management | recommended |
| Excalidraw | architecture / pipeline diagrams | optional |
| Omnisearch | fast full-text search | optional |

Keep active plugins under ~20 for performance.

## Git rhythm
- **Vault repo:** aggressive auto-commit (Obsidian Git) is fine — notes are messy by nature.
- **Code repo:** commit manually and deliberately. Don't share the autocommit habit across both.

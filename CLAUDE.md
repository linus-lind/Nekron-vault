# CLAUDE.md — Nekron Vault

Rules for Claude when working in this repo. Keep this file **lean and curated** — it loads every session.

## What this repo is
This is the **Obsidian vault** for **Nekron**, an ML-driven investing project (solo dev: linus-lind).
It is the **knowledge / decision / index layer** — NOT the code.

- Code / app repo: `github.com/linus-lind/Nekron` (separate; has its own CLAUDE.md)
- This vault repo: `github.com/linus-lind/Nekron-vault`
- Experiment tracker: **MLflow** (metrics source of truth; notes link out via `mlflow_run_id`)

## Golden rules
1. **Notes hold pointers + interpretation**, not artifacts. Code → git SHA, metrics → MLflow, data → storage. The vault holds the *why*.
2. **No secrets, ever** — API keys, broker creds, `.env`. Not in notes, not in examples.
3. **No code or data files** in the vault (no `.py`, no parquet/CSV market data).
4. **Every note gets frontmatter** with a `type`. No `type` → it won't show in dashboards.
5. **Reproducibility contract:** every `experiment`/`backtest` note must carry a `git_commit`.
6. **Links > folders.** Prefer `[[wikilinks]]` between model ↔ experiment ↔ backtest ↔ idea.

## Where things go
`00_Inbox` capture · `01_Daily` log · `02_Research` (Papers/Concepts/Ideas) · `03_Data` (Sources/Features) · `04_Models` registry · `05_Experiments` (`HPO/` = tuning campaigns) · `06_Backtests` · `07_Live` (Runbooks/Incidents/Journal) · `08_Decisions` ADRs · `09_Ops` infra · `_templates` · `_dashboards` · `_meta` conventions.

## Conventions (authoritative details in `_meta/`)
- Use the exact status vocabularies and metric formats in `_meta/Frontmatter Schema.md`.
- Metrics are **raw numbers**, no `%` or quotes (`max_drawdown: -0.18`, not `"-18%"`).
- Naming rules in `_meta/Vault Conventions.md` (e.g. `exp-YYYYMMDD-<model>-<short>`).
- New notes should be created from `_templates/` (Templater syntax `<% %>`).
- Hyperparameter searches follow `_meta/HPO Protocol.md`. Its sections 1-6 are pre-registered per campaign and are **never edited after the first trial** — amendments get a new `protocol_version` and apply forward only.

## GitHub bridge
Templates build clickable links to `github.com/linus-lind/Nekron/commit/<sha>` and `/blob/<sha>/<config>`. Keep these pointing at `linus-lind/Nekron`.

## Style
- **No emojis.** Not in responses, not in notes, templates, or dashboards created here.

## Git rhythm
- **This vault:** frequent/auto-commit is fine (notes are messy by nature).
- **Code repo:** commit deliberately — don't carry the autocommit habit over.

## Maintaining this file
Update when a new durable convention is agreed (data vendor, retrain cadence, naming rule). Keep it short; put long-form detail in `_meta/`, not here.

# Nekron Vault

Knowledge, research, and decision layer for **Nekron** — an ML-driven investing application.

This is an [Obsidian](https://obsidian.md) vault (plain Markdown). It is the *narrative and index* layer that sits beside the code:

- **Code / app:** `github.com/linus-lind/Nekron` (separate repo)
- **This vault:** knowledge, experiment logs, backtests, decisions, live ops
- **Experiment tracker:** MLflow (metrics source of truth; notes link out to runs)

## What lives here (and what doesn't)

| Belongs here | Does NOT belong here |
|---|---|
| Strategy ideas & hypotheses | Python code (that's the code repo) |
| Experiment & backtest write-ups | Raw market data / parquet / CSV |
| Model registry & decisions (ADRs) | Secrets, API keys, credentials |
| Live ops runbooks & incident logs | Authoritative metrics (that's MLflow) |
| Research notes, data-source docs | Large binary artifacts |

Notes hold **pointers** (git commit SHA, MLflow run URL, config path) plus your **interpretation**. Heavy artifacts stay in their proper systems.

## Structure

```
00_Inbox/       Fast capture, triage later
01_Daily/       Daily working log
02_Research/    Papers · Concepts · Ideas
03_Data/        Sources · Features
04_Models/      Model registry (one note per model/version)
05_Experiments/ One note per training run
06_Backtests/   One note per backtest
07_Live/        Runbooks · Incidents · Journal
08_Decisions/   Architecture/strategy decision records (ADRs)
09_Ops/         Infra, RunPod, cloud, cost tracking
_templates/     Templater templates
_dashboards/    Dataview live dashboards
_meta/          Conventions & frontmatter schema
_attachments/   Images, diagrams
```

## Setup

1. Install community plugins: **Dataview**, **Templater**, **QuickAdd**, **Obsidian Git**.
2. In Templater settings, set the template folder to `_templates`.
3. GitHub links point to `github.com/linus-lind/Nekron`.
4. See `_meta/Vault Conventions.md` and `_meta/Frontmatter Schema.md`.

Start at [[Start Here]].

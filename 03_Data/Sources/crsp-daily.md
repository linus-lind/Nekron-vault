---
type: data-source
provider: "CRSP"
asset_class: "US equities"
cost: ""
status: active        # active | evaluating | dropped
grain: "daily, (date, entity)"
coverage_start: 1990-01-02
coverage_end: 2025-12-31
created: 2026-09-15
tags: [data]
---

# crsp-daily

**Provider:** `= this.provider` · **Asset class:** `= this.asset_class` · **Status:** `= this.status` · **Grain:** `= this.grain`

CRSP daily stock file, CIZ layout. The single source behind every model in the repo so far.

## Access
One local CSV at `data/crsp.csv` in the code repo (gitignored). ~39.7 GB as of 2026-09-15. Read through `configs/data/crsp.yaml`, which pins the column subset and dtypes; nothing reads the file directly.

## Coverage
`= this.coverage_start` to `= this.coverage_end`, daily. Entity-major on disk: all dates for one `PERMNO`, then the next. That ordering is why ingestion is chunked rather than streamed by date.

## Schema
Identity `PERMNO` / `PERMCO`, `Ticker`, `SICCD`. Dates appear twice — `YYYYMMDD` as an integer and `DlyCalDt` as `DD/MM/YYYY`; the config selects one and the format is not negotiable.

Prices and returns: `DlyOpen` / `DlyHigh` / `DlyLow` / `DlyClose` / `DlyPrc`, `DlyBid` / `DlyAsk`, `DlyRet` / `DlyRetx`, `DlyCap`, `DlyVol`, `DlyPrcVol`, `ShrOut`, `DlyFacPrc`. Market aggregates travel on every row: `vwretd`, `vwretx`, `ewretd`, `ewretx`, `sprtrn`.

## Gotchas (critical for quant)
- **Survivorship:** the file carries delisted securities (`DelActionType`, `DelReasonType`, `SecurityEndDt`), so the universe is point-in-time *if* the filter is applied per date. The `top_n_market_cap` filter does exactly that.
- **Corporate actions:** handled in preprocessing via `cumulative_factor` and `corporate_adjustment` from `DlyFacPrc`. Raw prices are not adjusted in the file.
- **Point-in-time:** `SecInfoStartDt` / `SecInfoEndDt` bound when a row's security metadata was valid. Header fields (`HdrCUSIP`) are current-as-of-extract and are not point-in-time — do not use them as features.
- **Missing returns:** `DlyRetMissFlg` and `DlyPrcFlg` mark imputed or stale values. The preprocessing pipeline filters on them; a feature built before that stage would inherit them.
- **Restatements:** a re-extract can change historical rows. That is what `data_key` in a run's parameters exists to detect — it moves when the file does.

## Used by
- [[cae-v1]] — beta and factor inputs, target `fwd_ret_1d`
- [[hpo-20260909-cae-v1]] — tuning campaign, `1990-01-01 .. 2019-12-31`

## Reserved
**2020-01-01 onward is held back untouched** for the end-to-end evaluation of the finished strategy. No model config may ingest it, and no tuning campaign may read it. Check `selection.end_date` before starting anything.

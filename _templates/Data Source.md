---
type: data-source
provider: "<% tp.system.prompt('Provider (e.g. Polygon, IBKR)') %>"
asset_class: "<% tp.system.prompt('Asset class') %>"
cost: ""
status: active        # active | evaluating | dropped
grain: "<% tp.system.prompt('Grain (e.g. daily, (date, entity))') %>"
coverage_start:       # YYYY-MM-DD, the first date the source actually holds
coverage_end:
created: <% tp.date.now("YYYY-MM-DD") %>
tags: [data]
---

# <% tp.file.title %>

**Provider:** `= this.provider` · **Asset class:** `= this.asset_class` · **Status:** `= this.status` · **Grain:** `= this.grain`

## Access
Endpoint / SDK, auth method (key stored in ___, NOT here), rate limits.

## Coverage
`= this.coverage_start` to `= this.coverage_end`. State any range deliberately reserved and not to be read.

## Schema
Fields, frequency, timezone of timestamps, adjustment conventions.

## Gotchas (critical for quant)
- Survivorship bias?
- Look-ahead / point-in-time correctness?
- Restatements / revisions?
- Corporate actions handling?
- Gaps / missing data?

## Cost & limits
`= this.cost`

## Used by
Features / models that depend on this feed.

---
type: paper
aliases: [Crossformer]
authors: "Yunhao Zhang, Junchi Yan"
year: 2023
url: "https://openreview.net/forum?id=vSVLM2j9eie"
rating: 0
status: read
created: 2026-08-18
tags: [paper, research, time-series, transformer]
---

# Crossformer - Cross-Dimension Dependency

*Crossformer: Transformer Utilizing Cross-Dimension Dependency for Multivariate Time Series Forecasting* (ICLR 2023, OpenReview:vSVLM2j9eie).
Domain: multivariate time series forecasting.

## Method
Crossformer explicitly models both **cross-time** and **cross-dimension (cross-variate)** dependency, and does so at multiple scales, unlike prior Transformers that attend over time only. [[Modelling multivariate correlations]]

- **Dimension-Segment-Wise (DSW) embedding** — each variate series is split into equal-length segments; each segment is linearly projected to a vector and given a position embedding, producing a 2D array of tokens (variate x segment). [[Patching (time series)]]
- **Two-Stage Attention (TSA)** — self-attention is applied in two stages: first across time (over segments within each variate), then across variates. The cross-variate stage uses a **router mechanism** (a small set of router tokens gather then distribute) to avoid quadratic cost in the number of variates.
- **Hierarchical Encoder-Decoder (HED)** — adjacent segments are merged between layers to build coarser scales, so the encoder runs fine to coarse. The decoder combines the multi-scale representations for the final forecast.

## Procedure
```
multivariate series  (D variates, T steps)
  -> DSW embedding: segment each variate, linear-project each segment,
     add position embedding                      -> 2D token array (D x L)
  -> Two-Stage Attention:
        (1) cross-time      MSA over segments within each variate
        (2) cross-dimension MSA over variates (router mechanism)
  -> merge adjacent segments (linear projection)  -> coarser scale
  -> Two-Stage Attention again ... repeat         (fine -> coarse)
  -> decoder aggregates multi-scale representations
  -> linear projection                            -> forecast
```

## Relevance
- Multivariate forecasting with **explicit cross-variate dependency** plus **multi-scale (fine-to-coarse)** representations — relevant to modelling correlations across assets/features at several horizons.

## Further reading
- [[FEDformer]] — proposes that time series have a sparse representation in the frequency domain and builds a frequency-enhanced transformer. Potentially interesting direction.

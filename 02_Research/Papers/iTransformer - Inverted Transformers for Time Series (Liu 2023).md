---
type: paper
authors: "Yong Liu, Tengge Hu, Haoran Zhang, Haixu Wu, Shiyu Wang, Lintao Ma, Mingsheng Long"
year: 2023
url: "https://arxiv.org/abs/2310.06625"
rating: 0
status: read
created: 2026-08-18
tags: [paper, research, time-series, transformer]
---
Tokenizing financial time series
# iTransformer - Inverted Transformers for Time Series

*iTransformer: Inverted Transformers Are Effective for Time Series Forecasting* (arXiv:2310.06625, ICLR 2024).
Domain: multivariate time series forecasting.

## Method
- **Motivation (Fig. 2)** — a standard temporal token embeds all variates at a single timestamp, fusing distinct physical measurements and potentially delayed events. With an effectively single-step receptive field, such tokens can fail to convey distinguishable information and produce meaningless attention maps.

![[iTransformer-fig2-comparison.svg]]

- **Inversion** — iTransformer swaps the temporal and feature dimensions: each variate's whole sequence is embedded into a **variate token**, rather than forming a temporal token over the multivariate feature vector. [[Variate tokens]]
- **Cross-variate attention** — multi-head self-attention is applied across variate tokens, capturing multivariate correlations. [[Modelling multivariate correlations]]
- **Shared feed-forward network** — an MLP applied independently to each variate token learns series representations. This inherits the efficiency of linear predictors and encourages the model to capture intrinsic series properties (amplitude, periodicity, frequency).
- **LayerNorm on variate tokens** — applying LayerNorm per variate token (rather than across variates of a timestamp) avoids mixing independent series distributions and the resulting oversmoothing. Normalizing tokens toward a Gaussian also lets cross-variate attention be interpreted as computing correlations. [[LayerNorm]]
- **Positioned against related adaptations** worth going deeper on: [[Stationarization]] (explicit non-stationary handling), [[Channel independence]], [[Patching (time series)]], and [[Crossformer]] (cross-time and cross-variate dependencies).

## Procedure
```
multivariate series X   (T time steps, N variates)
  -> invert: embed each variate's full series   ->  N variate tokens
  -> multi-head self-attention across variate tokens   (multivariate correlations)
  -> add & LayerNorm (per variate token)
  -> shared feed-forward network per variate token     (series representation)
  -> add & LayerNorm
  -> linear projection  ->  forecast horizon per variate
```

## Relevance
- Time series modelling and prediction.
- Modelling of multivariate correlations.

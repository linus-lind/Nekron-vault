---
type: paper
aliases: [PatchTST]
authors: "Yuqi Nie, Nam H. Nguyen, Phanwadee Sinthong, Jayant Kalagnanam"
year: 2022
url: "https://arxiv.org/abs/2211.14730"
rating: 0
status: read
created: 2026-08-18
tags: [paper, research, time-series, transformer]
---

# PatchTST - Patching and Channel Independence

*A Time Series is Worth 64 Words: Long-term Forecasting with Transformers* (arXiv:2211.14730, ICLR 2023).
Domain: long-term multivariate time series forecasting.

## Method
PatchTST combines two design choices: **patching** and **channel independence**.

- **Patching** — each series is segmented into subseries-level patches that serve as input tokens. Three-fold benefit: retains local semantic information in the embedding, reduces attention computation/memory quadratically for a given look-back, and lets the model attend to a longer history. [[Patching (time series)]]
- **Channel independence** — each channel (variate) is a single univariate series, and all channels share the same embedding and Transformer weights but are processed **independently**. Argued to reduce overfitting and improve out-of-sample performance versus channel-mixing models; the trade-off is that cross-variate dependencies are ignored by design. [[Channel independence]] [[Modelling multivariate correlations]]
- **Self-supervised pretraining** — a masked-patch objective (predict masked patches) yields representations that fine-tune well and can outperform supervised training on large datasets.

## Procedure
```
per variate (channel), independently, with shared weights:
  series -> split into subseries-level patches
        -> linear-project patches (+ position embedding)  -> patch tokens
        -> multi-head self-attention + FFN over patches
        -> linear head -> forecast
(all channels reuse the same backbone; no cross-channel attention)
```

## Relevance
- Local context matters: patching (local semantic embedding) again enhances Transformer forecasting.
- Channel independence reduces overfitting and is data-efficient, but disregards cross-variate dependencies — the opposite design choice to [[Crossformer]] and [[iTransformer - Inverted Transformers for Time Series (Liu 2023)|iTransformer]].

See [[Tokenizing financial time series]].

---
type: paper
aliases: [StableToken]
authors: "Yuhan Song, Linhao Zhang, Chuhan Wu, Aiwei Liu, Wei Jia, Houfeng Wang, Xiao Zhou"
year: 2025
url: "https://arxiv.org/abs/2509.22220"
rating: 0
status: read
created: 2026-08-18
tags: [paper, research, quantization, tokenization, robustness]
---

# StableToken - Noise-Robust Speech Tokenizer

*StableToken: A Noise-Robust Semantic Speech Tokenizer for Resilient SpeechLLMs* (arXiv:2509.22220, ICLR 2026).
Domain: semantic speech tokenization (noise-robust, for SpeechLLMs).

## Method
A consensus-driven tokenizer that makes discrete tokens stable under input perturbations. The quantizer (Voting-LFQ) extends [[Lookup-Free Quantization (LFQ)]] with a differentiable bit-level majority vote. [[Binary Spherical Quantization (BSQ)]]

- **Multi-branch quantization** — instead of a single quantizer, the encoded hidden state is linearly projected `n` times independently, and each projection is quantized independently (sign-based binary quantization). [[Straight-Through Estimator]]
- **Bit-wise majority vote** — the branch bit-vectors are averaged/summed and the sign is taken per bit (`B_final = sign(Σ_i b_i)`). With an **odd** number of branches `n`, this enforces a **strict majority rule**, correcting minority-branch errors.
- **Multi-view perturbation** — during training, a minority subset of `k < n/2` branches receive a perturbed hidden state while the `n−k` majority receive the clean one, boosting robustness while keeping the majority stable.
- **Consensus loss** — an MSE supervision term pulls each branch's projected vectors toward their mean (a stable reference), providing explicit intermediate supervision and enforcing consensus.

## Procedure
```
encoder hidden state h
  -> n independent linear projections            v_1 ... v_n   (odd n)
  -> per-branch binary quantization (LFQ, sign)  b_i = sign(v_i)
  -> bit-wise vote:  s = sum_i b_i ;  B_final = sign(s)   (strict majority)
  -> single stable token sequence

Training (noise-aware consensus):
  - clean h -> n-k majority branches ;  perturbed h' -> k<n/2 minority branches
  - consensus loss (MSE) pulls branch projections toward their mean/reference
```

## Relevance
The idea: learn a **latent stock representation** from noisy observations (e.g. closing prices). The latent representation is used to compute **residuals**, which ideally capture **idiosyncratic risk** — mean-reverting and statistically arbitrageable. A noise-robust, consensus-driven quantizer is attractive here because market inputs are extremely noisy, and a stable discrete representation could make the residual signal cleaner.

See [[Latent stock representation]], [[Idiosyncratic risk]], [[Statistical arbitrage]], [[Tokenizing financial time series]].

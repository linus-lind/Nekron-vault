---
type: paper
authors: "Yue Zhao, Yuanjun Xiong, Philipp Krähenbühl"
year: 2024
url: "https://arxiv.org/abs/2406.07548"
rating: 0
status: read
created: 2026-08-18
tags: [paper, research, quantization, tokenization]
---

# BSQ - Binary Spherical Quantization

*Image and Video Tokenization with Binary Spherical Quantization* (arXiv:2406.07548).
Domain: image/video tokenization, reconstruction, compression, and synthesis (BSQ-ViT).

## Method
BSQ is a [[Vector quantization|quantizer]] for tokenization with the following properties:

- **Implicit codebook** — no learned parameters for the codebook.
- **Codebook scales exponentially** with the number of spherical dimensions L (2^L codewords).
- **Bounded quantization error** — inputs are L2-normalized onto the unit hypersphere before and after binary quantization, so both points have unit norm and the error is bounded. [[Lookup-Free Quantization (LFQ)]] omits this normalization and therefore has unbounded error.
- **Factorized entropy loss** — the entropy loss uses a factorized (per-dimension independent) approximation to the marginal entropy, which yields an upper bound on the true entropy loss. [[Entropy loss (codebook usage)]]
- **Straight-Through Estimator (STE)** makes the binary step differentiable: `x_q = x + sg(sign(x) - x)`, where `sg` is stop-gradient. The forward pass is `sign(x)`; the gradient passes straight through (identity). [[Straight-Through Estimator]]

## Procedure
```
embedding z
  -> linear projection to L dimensions   v
  -> L2-normalize onto unit hypersphere  u = v / ||v||_2
  -> binary quantization                 b = sign(u) / sqrt(L)   (STE for gradients)
  -> inverse: dequantize -> inverse projection back to embedding dimension
```

## Relevance
- **Quantization to reduce noise in training** — discretizing continuous inputs collapses small perturbations onto the same code, filtering noise before it reaches the model.
- **Discrete codebook for stable representation learning** — a fixed, bounded set of codewords gives a more stable target space than raw continuous embeddings.


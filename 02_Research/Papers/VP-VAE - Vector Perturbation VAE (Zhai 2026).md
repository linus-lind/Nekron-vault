---
type: paper
aliases: [VP-VAE]
authors: "Linwei Zhai, Han Ding, Mingzhi Lin, Cui Zhao, Fei Wang, Ge Wang, Wang Zhi, Wei Xi"
year: 2026
url: "https://arxiv.org/abs/2602.17133"
rating: 0
status: read
created: 2026-08-18
tags: [paper, research, quantization, vae, representation-learning]
---

# VP-VAE - Vector Perturbation VAE

*VP-VAE: Rethinking Vector Quantization via Adaptive Vector Perturbation* (arXiv:2602.17133, 2026).
Domain: vector quantization / representation learning (VQ-VAE training).

## Method
VP-VAE **decouples codebook learning from encoder training** to avoid the pathologies of standard [[Vector quantization|VQ-VAE]] training. [[Codebook collapse]]

- **Problem** — standard VQ-VAE couples representation learning with codebook optimization. Surrogate gradients ([[Straight-Through Estimator]]) create a mismatch (non-differentiable forward, continuous backward), and concurrent codebook updates give non-stationary targets: the encoder must map to codebook entries while entries chase the shifting encoder distribution. This drives instability and **codebook collapse** (dead codes).
- **Key idea (codebook-free training)** — to the decoder, quantization is just a bounded, local **perturbation** of the latent (the quantization error). So instead of quantizing during training, perturb each latent to mimic the inference-time quantization error, training the encoder/decoder under inference-like conditions.
- **Scale alignment (adaptive radius)** — approximate the k-means quantization-error scale by viewing the latent space as `K` equiprobable Voronoi cells; the distance to the `M`-th nearest neighbor (`M ~ |S|/K`) estimates the local cell radius, adapting to local density and codebook size.
- **Distribution consistency (Metropolis-Hastings)** — generate perturbations with an MH transition whose **stationary distribution matches the empirical latent density**, so perturbed latents stay high-probability and the latent distribution remains stationary (unlikely configurations are rejected).
- **Objective** — reconstruction loss + a **normalization loss** regularizing each latent dimension to mean 0, variance 1.
- **Codebook (post-training)** — run the encoder over the training set, then K-Means (K-Means++) to obtain `K` centroids as the codebook; at inference each latent is quantized to its nearest centroid.
- Also derive **FSP (Finite Scalar Perturbation)**, a lightweight variant under an approximately-uniform latent assumption.

## Procedure
```
Training (codebook-free):
  encoder -> latent z
  scale:   M-th nearest-neighbor distance ~ local Voronoi radius   (equiprobable cells, M ~ |S|/K)
  perturb: z' = MH-sample around z   (stationary dist = empirical latent density)
  decoder reconstructs from z'
  loss = reconstruction + normalization (per-dim mean 0, var 1)

Codebook (post-training):
  collect latents over training set -> K-Means(++) -> K centroids = codebook

Inference:
  quantize each latent to nearest centroid
```

## Relevance
- Codebook-free, collapse-resistant quantization for learning stable discrete latent representations — connects to the quantization thread ([[Binary Spherical Quantization (BSQ)]], [[StableToken]]) and to [[Tokenizing financial time series]].

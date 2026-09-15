---
type: paper
aliases: [Peri-LN]
authors: "Jeonghoon Kim, Byeongchan Lee, Cheonbok Park, Yeontaek Oh, Beomjun Kim, Taehwan Yoo, Seongjin Shin, Dongyoon Han, Jinwoo Shin, Kang Min Yoo"
year: 2025
url: "https://arxiv.org/abs/2502.02732"
rating: 0
status: read
created: 2026-08-18
tags: [paper, research, transformer, normalization]
---

# Peri-LN - Normalization Placement in Transformers

*Peri-LN: Revisiting Normalization Layer in the Transformer Architecture* (arXiv:2502.02732, ICML 2025).
Domain: Transformer architecture / normalization placement, studied up to 3.2B-parameter LMs.

## Method
Compares three placements of [[LayerNorm]] relative to each sub-layer (module), analyzing forward/backward statistics beyond initialization.

- **Post-LN** — normalization applied after the residual addition. Can degrade gradient flow in deeper networks, causing vanishing gradients and slower/less stable convergence.
- **Pre-LN** — normalization applied to the module input (`x + Module(Norm(x))`). Improves gradient propagation and stabilizes early training, but hidden-state variance grows with depth: **linearly at initialization, but this does not persist during training**, where it can grow **exponentially** into "massive activations" (numerical instability, overflow risk).
- **Peri-LN** — normalization applied **both before and after** each module (`x + Norm(Module(Norm(x)))`). Strikes a balanced variance growth (not exponential) while preserving steady gradient flow, avoiding both vanishing gradients and massive activations. Already adopted by open-source models (Gemma 2, OLMo 2).

## Procedure
```
Post-LN:  x_{l+1} = Norm( x_l + Module(x_l) )
Pre-LN:   x_{l+1} = x_l + Module( Norm(x_l) )
Peri-LN:  x_{l+1} = x_l + Norm( Module( Norm(x_l) ) )   # LN on both module input and output
```

## Relevance
- Normalization-placement choice for Nekron's Transformer models: Peri-LN offers more stable variance growth and gradient flow at depth/scale, which matters if models get deep or training is unstable.


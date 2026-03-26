---
name: Model Collapse Research Project
description: NeurIPS 2026 paper on synthetic data growth rate and model collapse; General Theorem + M+V framework; Exp1 done, Exp2-3 pending
type: project
---

NeurIPS 2026 submission investigating when synthetic data improves or degrades model performance under exponential growth with a verifier.

**Core Research Question:** How must synthetic data growth rate k and verifier quality q interact to prevent model collapse?

---

## Theoretical Framework (Updated)

### General Theorem — Stability-Collapse Dichotomy (Theorem 1)

**Setup (M+V Framework):**
- Verifier V_q produces labels: ŷ = x⊤(qθ* + (1−q)θ_{t−1}) + ε
- Exponential weights: w_t = k^t, synthetic volume capped at n_s
- Ω = population covariance, M_∞ = tr(Ω⁻¹) = effective model complexity

**Key Lemmas:**
- Lemma 1: γ_t = k^t / Σk^i → c(k) = (k−1)/k  (universal, model-independent)
- Lemma 2: Variance injection per round → c²σ²Ω⁻¹/n_s (constant, due to n_s cap)

**Main Result:**
$$\mathrm{MSE}_\infty(q,k) = \frac{M_\infty \cdot \sigma^2 \cdot c(k)}{q \cdot (2 - q \cdot c(k)) \cdot n_s}$$

**Stability condition:**
$$q \geq q^*(k) = \frac{M_\infty \cdot \sigma^2 \cdot (k-1)}{2 \cdot \Delta \cdot k \cdot n_s}$$

**Key insight:** c(k) = (k−1)/k is universal across all model classes; only M_∞ is model-specific.

**Root cause of collapse:** finite compute (n_s cap) + exponential weighting. Without cap, MSE_∞ = 0 for any q > 0.

### Corollary 1 — Linear Regression
Ω = I_p, M_∞ = p → q*(k) = pσ²(k−1)/(2Δkn_s)

### Corollary 2 — VAE (planned)
M_∞ = tr(Σ_data⁻¹), same stability form

---

## Positioning vs. Related Work

| Paper | Gap |
|-------|-----|
| Yi et al. 2025 | Shows convergence to θ_c but no growth rate k analysis, no q*(k) |
| Feng et al. 2025 | Phase transition at p* but single-round, no k-dependence |
| Gerstgrasser 2024 | Only constant growth, no verifier |
| **Ours** | q*(k) derived; tight across 3 model classes |

---

## Experiment Progress

- **Exp1 (Linear Regression):** Complete. Five conditions validate Σ(fⱼ/Fⱼ)² criterion.
- **Exp1b (Phase Diagram):** Complete. (k,q) heatmap matches q*(k) theory precisely. See `exp1b_heatmap.png`.
- **Exp2 (VAE):** Planned. Discriminator as verifier (top-q% filtering); (k,q) heatmap with FID metric; MNIST.
- **Exp3 (LM):** Planned. Perplexity-based verifier filter.

## Key Files
- `/fast/smchoi/collapse_exp/exp1b_phase.py` — Exp1b implementation
- `/fast/smchoi/collapse_exp/exp1b_heatmap.png` — Phase diagram (validates theory)
- `/fast/smchoi/collapse_exp/paper_overview_1.md` — Full paper overview with General Theorem

## Working Directory
`/fast/smchoi/collapse_exp`

**Why:** Paper aims to formalize collapse conditions as a function of growth rate, providing the first joint (k, q) characterization.

**How to apply:** Focus on General Theorem + M+V framework. Exp2 uses discriminator verifier with discriminator score threshold as q. Theory boundary q*(k) should be validated across all three model classes.

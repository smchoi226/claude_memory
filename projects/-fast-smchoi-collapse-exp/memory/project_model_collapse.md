---
name: Model Collapse Research Project
description: NeurIPS 2026 paper on synthetic data growth rate and model collapse; M+V framework; Exp1/1b/verifier done, VAE(Exp2) verifier 설계 확정, LM(Exp3) 계획 확정
type: project
---

Synthetic data의 증가 속도와 model collapse의 정량적 관계를 밝히는 NeurIPS 2026 논문.

**Core Philosophy:**
> Synthetic data 증가 속도가 collapse를 일으키고, 오직 충분히 강한 verifier만이 이를 막을 수 있으며, 그 "충분히 강한"의 기준이 growth rate의 universal한 함수 c(k)로 결정된다.

**Why:** 기존 연구 (Gerstgrasser, Feng, Yi-Liu)는 "verifier가 도움이 된다"만 보였고, "growth rate k에서 정확히 얼마나 강한 verifier가 필요한가"를 closed-form으로 답하지 못했음.

---

## 논문 구조 (3개의 질문)

- **Q1.** 언제 collapse가 일어나는가? → Σ(fᵢ/Fᵢ)² criterion (linear reg formal; general mechanistic)
- **Q2.** Verifier가 얼마나 강해야 하는가? → q_eff ≥ q*(k) = M_∞σ²·(k-1)/(2Δkn_s)
- **Q3.** 현실적 verifier (D_real만)로 가능한가? → Score-based verifier, τ*(k) open problem

**Universal structure:** q*(k) = c(k) · M_∞σ²/(2Δn_s), c(k)=(k-1)/k는 모델 무관

---

## Theoretical Framework (M+V)

- Verifier: score function s(x,y) ≈ log p_real, D_verifier에서 추정 (θ* 불필요)
- q_eff := (E[y_sel|x] - E[y_synth|x]) / (E[y_real|x] - E[y_synth|x]) — model-agnostic
- Scalar q는 linear regression에서의 special case
- Theorem 1: MSE_∞ = M_∞σ²c(k) / (q_eff(2-q_eff·c(k))·n_s) [linear reg formal]
- Biased verifier: MSE_∞ ≥ σ² + δ² (irreducible floor)
- Finite compute (n_s cap)이 collapse의 근본 원인

---

## Score-based Verifier (확정된 설계)

세 모델 모두 동일 구조: **D_verifier로 학습한 reference model의 score → top-τ selection**

| Model | Score s(x, y) | Reference 학습 |
|-------|--------------|----------------|
| Linear | `-(y - x^T θ_ref)²` | OLS on D_verifier |
| VAE | `-‖x - recon_ref(x)‖²` | VAE on D_verifier |
| LM | `-PPL_ref(y)` | LM on D_verifier |

**Why:** discriminator 방식은 레퍼런스 없고 linear와 구조적 일관성 없음. Reference model reconstruction error가 linear regression의 residual-based score와 완벽히 대응함.

**How to apply:** exp_vae.py의 discriminator verifier를 reference VAE reconstruction error로 교체해야 함.

---

## Experiment Progress

- **Exp1 (Linear Reg, 5 conditions):** ✅ Complete. Σ(fᵢ/Fᵢ)² criterion 검증.
- **Exp1b (Phase Diagram k×q):** ✅ Complete. q*(k) boundary 이론과 정확 일치. `exp1b_heatmap.png`
- **Exp_verifier A (score-based k×τ):** ✅ Complete. τ<1.0 → k 무관 MSE≈1.3 안정. `exp_verifier.png`
- **Exp_verifier B (biased verifier q×δ):** ✅ Complete. MSE floor = σ²+δ² 이론 일치.
- **Exp2 (VAE):** 🔧 verifier 설계 확정, exp_vae.py 수정 필요 + crash 디버깅. Sherlock에서 실행 예정.
- **Exp3 (LM):** 📋 세팅 확정, 코드 미작성. Sherlock에서 실행 예정.

---

## Exp2 (VAE) 세팅 확정

- **데이터:** MNIST (Shumailov 2024와 동일)
- **모델:** MLP-VAE (latent=32)
- **Verifier:** Reference VAE (D_verifier로 학습) reconstruction error `-‖x - recon_ref(x)‖²`, top-τ selection
- **Grid:** k ∈ {1.5, 2.0, 2.5, 3.0, 3.5, 4.0} × τ ∈ {0.05, 0.10, 0.20, 0.30, 0.50, 0.70, 1.00}
- **T=25** rounds, N_REAL=3000, N_S=200, M=6, 3 seeds
- **Shumailov 참고:** 20 generations, 처음부터 재학습 방식

## Exp3 (LM) 세팅 확정

- **모델:** OPT-125m (Shumailov 2024와 동일), fine-tuning
- **데이터:** WikiText2 (Shumailov 2024와 동일)
- **Verifier:** Reference LM (D_verifier로 학습), score = `-PPL_ref(y)`, top-τ selection
- **Grid:** k × τ phase diagram (Exp2와 동일 구조)
- **T=10~15** rounds (Shumailov은 9 generations)
- **Metric:** Perplexity on original WikiText2 test set (+ tail n-gram entropy)

---

## 실행 환경

- **Sherlock** (Stanford HPC) — VAE, LM 실험 모두 여기서 실행
- SLURM 스크립트 작성 필요

---

## Key Files

- `/fast/smchoi/collapse_exp/exp1_linreg.py` — Exp1
- `/fast/smchoi/collapse_exp/exp1b_phase.py` — Exp1b
- `/fast/smchoi/collapse_exp/exp1b_heatmap.png` — Phase diagram
- `/fast/smchoi/collapse_exp/exp_verifier.py` — Score-based + biased verifier
- `/fast/smchoi/collapse_exp/exp_verifier.png` — Results
- `/fast/smchoi/collapse_exp/exp_vae.py` — VAE experiment (수정 필요: discriminator → reference VAE recon error)
- `/fast/smchoi/collapse_exp/paper_overview_2.md` — 최신 paper overview

## Related Papers (arxiv)

- Shumailov et al. Nature 2024 (2305.17493): VAE(MNIST, 20 gens) + LM(OPT-125m, WikiText2, 9 gens) + GMM. Verifier 없음, collapse 자체만 시연. 실험 방법론 참조.
- Yi, Liu, Cheng, Xu (2510.16657): verifier → θ_c 수렴; θ* oracle 가정; k 미분석
- Feng et al. ICLR 2025 (2406.07515): phase transition; single-round; k 없음
- Dohmatob et al. (2410.04840): strong collapse; random projection NN; verifier 없음
- Bakshi-Chakraborty (2602.10531): categorical distribution; sample size schedule

## Open Problems

- τ*(k) formal proof (truncated distribution + correlated rounds)
- Deep net M_∞ (NTK/Fisher 후보)
- VAE Exp2: exp_vae.py 수정 + crash 디버깅 + Sherlock 실행
- LM Exp3: 코드 작성 + Sherlock 실행

**How to apply:** Q1→Q2→Q3 서사 유지. Theorem 1은 linear reg formal proof; VAE/LM은 empirical validation + mechanistic argument. Score-based verifier가 scalar q의 현실적 구현임을 강조.

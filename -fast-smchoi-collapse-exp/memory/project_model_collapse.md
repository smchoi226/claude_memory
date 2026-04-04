---
name: model collapse / synthetic data 논문 프로젝트
description: NeurIPS 2026 제출 목표. Synthetic data growth rate와 verifier quality의 관계로 collapse 조건을 정식화하는 논문. 통신 실험 제거, 순수 ML theory + experiment.
type: project
---

## 목표
- NeurIPS 2026 (약 2달 후 마감) 1저자 제출
- Theory-heavy, experiment-light 포맷
- 통신(Sionna) 실험은 **이 논문에서 제거** (scope 명확화)

---

## 핵심 연구 질문

> **"Synthetic data의 growth rate와 verifier quality가 어떤 조건을 만족해야 모델 성능이 향상되는가?"**

기존 학계: "collapse를 막는 것"에 집중.
우리: collapse 조건을 growth rate의 함수로 정식화.

---

## 시스템 구조 (확정)

**M + V 구조 (G = M)**:
```
Real data D_real
     +
M → synthetic X̃  →  V (filter) → D_synth
     ↑                                 |
     └──── retrain on D_real + D_synth ┘
```
- M: 훈련하려는 모델 (= 생성기 G)
- V: 생성된 데이터 품질 판단하는 verifier
- Teacher-student (G≠M) 케이스는 이 논문에서 다루지 않음

---

## 핵심 프레임워크

**Growth rate fᵢ**: iteration i에서 추가되는 synthetic sample 수
**Fᵢ** = n₀ + Σⱼ≤ᵢ fⱼ (누적 총 sample 수)

**Theorem 1 (Growth Rate → Collapse)**:
```
E[test MSE] ≈ σ²(1 + C · Σⱼ₌₁ⁱ (fⱼ/Fⱼ)²)
```
- Σ(fⱼ/Fⱼ)² < ∞  →  stable
- Σ(fⱼ/Fⱼ)² → ∞  →  collapse

**Corollary**:
- fⱼ = 1 (Gerstgrasser): Σ ≤ π²/6 → stable ✓
- fⱼ = j (polynomial): Σ 수렴 → stable ✓
- fⱼ = kʲ (exponential): Σ ~ i·(k-1)² → ∞ → **collapse 필연** ✗

**Theorem 2 (Verifier Stability Condition)**:
```
E[test MSE] ≈ σ²(1 + C · Σⱼ (fⱼ/Fⱼ)² · (σ_eff(j)/σ)²)
```
Stability ↔ Σ (fⱼ/Fⱼ)² · σ_eff(j)² < ∞

지수 성장(fⱼ=kʲ) 하에서: fⱼ/Fⱼ → (k-1)/k = const
→ stability ↔ **σ_eff(j) → 0 필요** (perfect V 없으면 불가능)

**σ_eff**: verifier가 걸러낸 후 synthetic label에 남아있는 잔류 noise std.
- perfect V: σ_eff = 0
- imperfect V: σ_eff > 0 (상수이면 지수 성장 하에서 collapse)

---

## 현실 세계 연결

인터넷 AI-generated content: k≈3 (지수 성장)
→ 현재 인터넷 학습 패러다임은 필연적으로 collapse 방향
→ 막으려면: perfect V (수학/코드처럼 정답 명확한 domain) 또는 growth rate cap (정책적)

---

## 관련 논문 전체 landscape (2026-03-25 기준)

| 논문 | Linear reg? | V? | 핵심 | 우리와의 관계 |
|---|---|---|---|---|
| Shumailov 2024 (Nature) | ✗ | ✗ | Collapse 발견 | 출발점 |
| Gerstgrasser 2024 | ✓ | ✗ | Accumulation → 방지, fᵢ=1 가정 | 우리가 일반화 |
| Dohmatob 2024 (NeurIPS) | ✓ | ✗ | Scaling law 분석 | 이론 참고 |
| Strong MC (ICLR 2025) | ✓ | ✗ | Random projection 확장 | 방법론 참고 |
| Yi et al. 2510.16657 (v2: 2026-03-05) | ✓ | ✓ | V 있으면 방지, but θ_c로 수렴 (biased) | **핵심 경쟁작** |
| 2602.16065 (2026-02) | ✗ | ✗ | 오염 데이터서도 수렴, α 고정 가정 | 관련 있음 |
| 2602.14029 (2026-02) | ✓ | ✗ | Self-training U-shaped curve | 부분 관련 |

**Yi et al. 한계 (우리가 해결)**:
- growth rate 분석 없음 (nₖ linear 가정)
- "eventually plateaus and reverses" 관찰만, 수식 증명 없음
- θ_c ≠ θ* 수렴 → biased (이것도 우리 Theorem 2로 설명)

---

## 논문 구조 (확정)

```
1. Introduction
   - 인터넷 AI-generated content 지수 증가 동기
   - 기존: collapse 방지에 집중. 우리: 성능 향상/저하 조건
   - Contributions 4개

2. Related Work
   § Model Collapse (Shumailov, Dohmatob, Strong MC)
   § Avoiding Collapse via Accumulation (Gerstgrasser, 2602.16065)
   § Verification-based Approaches (Yi et al.)

3. Framework
   3.1 Setup: linear regression, M+V 구조, fᵢ 정의
   3.2 Theorem 1: growth rate → collapse condition
   3.3 Theorem 2: verifier stability condition
   3.4 Corollaries: 지수 성장 하에서 필요 조건

4. Experiments
   4.1 Exp 1: Linear regression (controlled) ← 완료
   4.2 Exp 2: VAE on MNIST
   4.3 Exp 3: Language model (optional)

5. Discussion & Conclusion
   - 현실 함의: k≈3 인터넷 성장률 → collapse 필연
   - 대책 3가지 (perfect V, real data 확보, growth cap)
```

---

## 실험 현황 (2026-03-25)

### Exp 1: Linear Regression ✅ 완료
- 경로: `/fast/smchoi/collapse_exp/exp1_linreg.py`
- 결과 이미지: `/fast/smchoi/collapse_exp/exp1_mse.png`, `exp1_diagnostic.png`
- 파라미터: p=20, n₀=300, k=3, T=25, 200 seeds
- 방법: sufficient statistics (A=X^TX, b=X^Ty) 추적 → 지수 성장도 메모리 문제 없음
- **결과 요약**:
  - 조건 A (fᵢ=1): MSE=1.08, stable ✓
  - 조건 C (fᵢ=i): MSE=1.11, stable ✓
  - 조건 D (fᵢ=kⁱ + perfect V): MSE=1.00, Bayes error까지 수렴 ✓
  - 조건 E (fᵢ=kⁱ + biased V): MSE=1.25, plateau (Yi et al. 재현) ✓
  - 조건 B (fᵢ=kⁱ, no V): MSE=1.65, 계속 상승 collapse ✓
- 이론과 완벽 일치. **논문 Figure 1, 2로 바로 사용 가능**

### Exp 2: VAE on MNIST ⬜ 미착수
### Exp 3: LM (optional) ⬜ 미착수

---

## 현재 상태 (2026-03-25)
- 프레임워크 확정
- 경쟁 논문 분석 완료 (Yi et al. v2 포함)
- Exp 1 완료, 이론과 일치
- 다음: Theorem 증명 초안 OR Exp 2 (VAE MNIST) 설계

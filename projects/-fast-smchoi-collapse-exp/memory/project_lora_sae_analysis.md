---
name: LoRA-SAE Feature Analysis Research
description: LoRA fine-tuning이 SAE feature 수준에서 무엇을 어떻게 변화시키는지 연구. Neural Anatomy 논문의 핵심 실험적 기반.
type: project
---

## 핵심 Research Question

> "LoRA fine-tuning은 어떤 SAE feature를 어떻게 변화시키는가?"

**Why:** 두 커뮤니티(mechanistic interp vs PEFT)가 완전히 따로 놀고 있음. 이 교차점에 연구가 거의 없음.

**How to apply:** Neural Anatomy 논문(biological hierarchy)의 핵심 실험 기반. "기관(LoRA) = 특정 조직(circuit)의 feature를 강화하는 도구"임을 실증.

---

## 핵심 Hypothesis

```
H1 (Sparsity):    LoRA는 소수의 feature만 크게 바꾼다
H2 (Specificity): Task A LoRA와 Task B LoRA가 바꾸는 feature가 다르다
H3 (Semantic):    변화한 feature들이 task와 의미적으로 연관된다
```

---

## 선행 연구 지도

| arXiv | 제목 | 관련도 | 핵심 |
|-------|------|--------|------|
| 2504.02922 | Crosscoder for Chat-Tuning (NeurIPS 2025) | **High** | Gemma 2B base vs it SAE feature 비교, 가장 직접적 방법론 |
| 2502.11812 | Fine-Tuning via Circuit Analysis (ICML 2025) | **High** | fine-tuning 후 circuit node 유지, edge 변화 → Circuit-Aware LoRA |
| 2410.21228 | LoRA vs Full FT: Illusion of Equivalence | **High** | LoRA가 "intruder dimensions" 생성 → forgetting 원인 |
| 2509.12934 | Anatomy of Alignment (FSRL) | **High** | SAE feature로 preference optimization 분해, LoRA ≈ feature steering 동등성 |
| 2603.00148 | Mechanistically Guided LoRA (2026) | **High** | SAE feature 식별 → LoRA 레이어 선택. 네 방향과 가장 유사한 최신 사례 |
| 2412.17626 | Tracking Feature Dynamics in LLM Training | **High** | training 중 SAE feature 추적 방법론 (SAE-Track) |

**핵심 Gap**: LoRA 특화 crosscoder 없음 (기존은 full fine-tuning만). LoRA rank vs feature 변화량 미탐구.

---

## 실험 계획

- 모델: Gemma 2 2B (Gemma Scope SAE 오픈소스)
- Task: Math (GSM8K) vs Code (CodeAlpaca)
- 방법: base vs LoRA-FT activation → SAE encode → delta 분석

## 코드 레포

https://github.com/smchoi226/SAE_Sparse_Autoencoder

파일: train_lora.py, lora_sae_analysis.py, run_lora.sbatch

## 특별 연결고리: Model Collapse 연구와의 접점

gradient fine-tuning(no replay) → collapse 시 어떤 SAE feature가 사라지는가는 완전히 미탐구.
LoRA frozen base = stable인 이유를 SAE feature 보존 관점에서 설명 가능.

## 실험 환경

- 실험 실행: Stanford Sherlock HPC
- 로컬 세팅: RTX 3090 (snu-4)

## 다음 할 일

- [ ] Sherlock에서 HuggingFace 로그인 후 LoRA 학습
- [ ] base vs LoRA SAE feature delta 시각화
- [ ] Crosscoder 방법론으로 feature 비교

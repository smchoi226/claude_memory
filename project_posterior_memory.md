---
name: posterior-memory 프로젝트 탐색
description: SNU-PI/posterior-memory GitHub 레포 탐색 및 개념 설명 대화 기록
type: project
---

## 레포 기본 정보
- 위치: /fast/smchoi/sionna/posterior-memory/
- GitHub: https://github.com/SNU-PI/posterior-memory
- SSH: smchoi226 계정, ~/.ssh/id_ed25519_github
- SNU-PI org 멤버 아님 → fork→PR 방식으로만 기여 가능

---

## 한 줄 요약

> "메모리(행렬 M)에 글씨를 어떻게 쓰면 기억도 잘 하고 수정도 잘 하나?"를 수학적으로 설계하는 연구.

---

## 배경 개념 (사용자와 정리한 것)

**Context window**: LLM은 처음부터 지금까지의 토큰을 전부 한 번에 입력으로 받음. Context window = 그 최대 길이 제한.

**Bounded context**: 이 프로젝트는 일부러 sliding window attention으로 최근 N토큰만 보게 막음. 왜냐면 full attention이면 메모리를 쓰는지 안 쓰는지 식별 불가 (Regime R1).

**외부 메모리 M**: M ∈ ℝ^{m×d} 행렬. 학습된 파라미터도 아니고 hidden state도 아닌, inference 중에 gradient로 업데이트되는 작은 RAM 같은 것. 에피소드 단위로 리셋됨.

**기존 메모리 접근들**:
- RAG: 외부 DB 검색 → context에 붙임. 읽기만 가능, 쓰기 없음. 현재 가장 실용적.
- KV Cache: 계산 캐시, 모든 LLM이 씀.
- Recurrent (LSTM/Mamba): hidden state가 메모리. 선택적 읽기 불가.
- Titans: ∇ℓ로 weight 업데이트. 새 정보 반영은 잘 하나 기존 기억 날림.
- NTM/Explicit slots: 명시적 행렬, 연구 단계 (실용화 안 됨).

**Explicit slots가 오랫동안 실용화 못 된 이유**: write rule이 문제. 이 논문의 가설: "write rule을 제대로 설계하면 됨."

---

## 동작 방식

**블록(block)**: N개 토큰 묶음. 토큰 하나하나가 아니라 블록 단위로 처리.

**읽기(read)**: 블록 처리 중 매 토큰마다 M에 cross-attention. 각 레이어마다 별도 reader.
```
각 토큰 hidden state (query) → M 슬롯들 (key/value) → cross-attention → residual에 더함
```

**쓰기(write)**: 블록 끝날 때 딱 한 번.
```
Block_t 처리 → NTP loss 계산 → M 업데이트 → Block_{t+1}부터 새 M 사용
```

**write-after-block 원칙**: 같은 블록으로 쓰고 읽으면 label leakage. 반드시 다음 블록부터 쓴 M을 사용.

**이 프로젝트는 생성 모델이 아님**: 토큰 생성이 아니라 NTP(다음 토큰 예측)로 M write rule을 학습/평가하는 구조.

---

## 핵심 Write Rule

$$M \leftarrow M + \eta \cdot A_t \left[ -\Lambda_t(M - \mu_t) - \nabla_M \ell_t \right]$$

- `−Λ(M−μ)`: retention prior. 과거 값에서 멀어지면 당겨오는 힘.
- `−∇ℓ`: NTP loss gradient. 예측 틀렸으면 그 방향으로 수정. (Titans와 동일한 항)
- 둘의 합력 = posterior score ascent
- NTM/Titans/RMT가 이 framework의 특수 케이스로 통합됨

---

## 데이터셋

**완전 합성(synthetic) 데이터.** 실제 텍스트 없음.

토큰 = 추상 심볼 (key0, val_30, QUERY, SEP 등).

**Task 1 (Belief Revision)**:
```
Block 0: key0=val30, key1=val서울, ...  (사실 주입)
Block 1~4: filler filler filler ...    (distractor)
Block 5: key0=val35, ...               (수정)
Block 6: QUERY key0=? QUERY key1=?    (쿼리)
```
수정된 건 새 값, 수정 안 된 건 원래 값으로 답해야 함.

**Task 2 (Selective Retention)**: 수정 블록 없이 순수 기억 유지만 테스트.

**왜 합성?**: "정답이 반드시 메모리를 통해서만 전달된다"는 보장이 필요. 실제 텍스트로는 불가능.
에피소드마다 key-value 매핑 랜덤 셔플 → 통계적 치팅 방지.

---

## 실험 구조 (각 실험의 근본 질문)

**EXP-000** ✅: 파이프라인이 그냥 돌아가나?

**EXP-001** ✅: Gradient로 M 쓰는 게 근본적으로 되나?
- pure_gradient vs controller_write(MLP) vs no_observation(gradient 없음)
- 결과: pure_gradient만 작동 (36.7%, chance의 22.9×). MLP는 chance, gradient 없으면 NaN 발산.
- 결론: **gradient 방향 자체가 핵심.**

**EXP-002** 🔄: 복잡한 write rule이 단순한 것보다 좋은가? (논문 핵심 주장 H1)
- pure_gradient → per_slot_lr → no_prior → full_model 단계별 검증
- `full_model > no_prior`가 나와야 prior(Λ항)가 정당화됨

**EXP-002b** 📋: Revision 잘 하게 됐는데 기억력은 망가지지 않았나?
- 순수 retention 태스크로 non-inferiority 확인

**EXP-003** 📋: 각 컴포넌트가 정말 필요한가? (논문용 풀 ablation, 8 variants × 5 seeds)
- causal intervention 포함: memory swap, slot zeroing, read-mask ablation

**EXP-004** 📋: 슬롯이 진짜 개별 주소처럼 동작하나?
- m = {4, 8, 16, 32} 변화시켜 기억 용량 비례 여부 확인

**결정 기준**:
- full_model > no_prior → H1 지지, 논문 진행
- full_model ≈ no_prior → 기여가 분석/통합뿐, 포지션 페이퍼로 피벗
- gradient 자체가 안 됨 → 전면 재검토 (EXP-001에서 이미 통과)

---

## 현재 상태 (2026-03-20)
- EXP-002 실행 중 (2× H200, 20 runs)
- 부분 결과: pure_gradient ~0.370 (EXP-001과 일관), per_slot_lr 학습 중
- Multi-agent 워크플로우: Codex + Claude agent들이 실험 진행 (reviews/ 폴더에 기록)

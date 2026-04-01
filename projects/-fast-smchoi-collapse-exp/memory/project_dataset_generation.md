---
name: Internet-Scale State-Action-Reasoning Dataset Project
description: Multi-domain (state, action, reasoning) triple dataset from internet sources — GitHub viral target, embodied AI + general reasoning pre-training
type: project
---

Goal: Build and publicly release a large-scale (state, action, reasoning) triple dataset from internet sources (YouTube commentary, game replays, board games, etc.), targeting GitHub viral + embodied AI / VLA community adoption.

**Core Novelty:** All existing large-scale datasets have ONLY (state, action) pairs — no reasoning:
- Open X-Embodiment: 100만+ trajectories, ❌ reasoning
- DROID: 76k trajectories, ❌ reasoning
- AgiBot World: 100만+ trajectories, ❌ reasoning
- Ego4D: 3,670h, ❌ reasoning
- D2E: 1,300h+, ❌ reasoning
- ECoT: 80k steps, ✅ reasoning (but robot data only, tiny scale)
**This gap is the key opportunity.**

**Key Theoretical Backing:**
- Policy(행동)는 도메인 간 전이 안 됨 (AlphaZero, MuZero = 각 게임 별도 학습)
- **Reasoning(사고 패턴)은 전이됨** — SPIRAL 논문(2506.24119): 게임 reasoning → 수학/추론 벤치마크 +10%
- D2E: GTA/Minecraft → 실제 로봇 manipulation 80% 성공
- 다양한 도메인 reasoning이 단일 도메인 대규모보다 범용 능력에 더 효과적일 수 있음

**Data Sources (well-defined state/action/reasoning):**

계층 1 — 완벽하게 정형화 (바로 수집 가능):
- 바둑: KGS/OGS API, 해설 영상 ASR → reasoning (기보는 저작권 없음)
- 체스: Lichess API (study + Stockfish 분석 + 해설), FEN/UCI 포맷
- 포커: GTO+ solver 분석, 전략 해설
- 퍼즐: 수독, 루빅스 큐브, 수학 문제 (Khan Academy 등)

계층 2 — 잘 정형화:
- StarCraft II: SC2 replay API (완벽한 게임 상태 추출)
- Dota 2: OpenDota API (1억+ 게임)
- GitHub PR: 코드 before/after + PR 설명 + 리뷰 코멘트
- LeetCode: 문제 → 풀이 단계 → editorial

계층 3 — 처리 필요:
- 게임 해설 YouTube 영상 (IDM pseudo-label + Whisper ASR → reasoning)
- 요리 영상, 스포츠 중계, 바둑/체스 해설

**핵심 아이디어 — 게임 해설 영상:**
유튜브 해설 영상 = 해설자가 자신의 행동을 실시간 설명 = 공짜 human reasoning
→ ASR(Whisper)으로 음성 전사 → 타임스탬프 정렬 → reasoning으로 사용
→ VLM 생성 reasoning보다 품질 높음, 비용 거의 0

**파이프라인:**
1. 소스별 수집 (YouTube yt-dlp / game API)
2. state 추출 (프레임 / 게임 상태)
3. action 추출 (IDM pseudo-label 또는 API에서 직접)
4. reasoning 추출 (ASR 전사 or Stockfish 분석 or VLM 생성)
5. 통합 스키마로 정규화
6. LeRobot 호환 포맷으로 HuggingFace Hub 공개

**통합 스키마:**
```json
{
  "domain": "chess",
  "state": "rnbqkbnr/pp1ppppp/...",
  "action": "e2e4",
  "reasoning": "Controls center, opens diagonal for bishop...",
  "source": "lichess_study_abc123",
  "quality_score": 0.92
}
```

**저작권 전략:**
- raw 영상/음성 배포 ❌
- (video_id, timestamp, transcript, extracted features)만 배포 ✅
- 재현 스크립트 공개 (LAION, HowTo100M 방식)
- 기보/게임 기록 자체는 저작권 없음 (fact) → 자유롭게 배포 가능
- 한국 저작권법 제35조의5: 비상업적 연구 목적 TDM 허용

**규모 목표:**
- 최소 바이럴 기준: 500k~1M triples
- "Internet-scale" 주장: 10M+
- ECoT(80k) 대비 100x 이상이 핵심 마케팅 포인트

**바이럴 전략:**
- LeRobot 호환 포맷 → HuggingFace 생태계 직접 타겟
- README에 도메인별 reasoning 예시 GIF
- arXiv 테크 리포트 동시 공개
- 논문 contribution: "다양한 reasoning이 전이된다" 실험 증명

**OWA Toolkit 연계:**
- D2E 팀(worv-ai)이 만든 오픈소스 데스크톱 캡처 툴킷
- ocap = Windows 11 + NVIDIA 전용, pip install ocap
- OWAMcap 포맷: video.mkv + session.mcap (이벤트 로그)
- 플러그인 시스템 (entry point 기반) → ASREvent 메시지 타입 추가 가능
- YouTube 기반이면 ocap 불필요, yt-dlp + Whisper로 충분

**Key Related Papers:**
- D2E (arXiv:2510.05684) — desktop video → robot transfer, no reasoning
- VPT (arXiv:2206.11795) — IDM pseudo-label paradigm (OpenAI)
- ECoT (arXiv:2407.08693) — reasoning for robot data (small scale, robot only)
- CoT-VLA (arXiv:2503.22020) — visual CoT for VLA
- LAPA (arXiv:2410.11758) — latent action pretraining from videos
- SPIRAL (arXiv:2506.24119) — multi-game reasoning → +10% on math/reasoning benchmarks
- Open X-Embodiment (arXiv:2310.08864) — standard robot dataset (no reasoning)

**경쟁 현황 (아무도 안 했음):**
- 로봇 trajectory 데이터셋 (Open X-Embodiment, ARIO): 멀티 도메인 ✅, reasoning ❌
- 수학/코드 reasoning 데이터셋 (Open-Thoughts, Sky-T1): reasoning ✅, embodied ❌
- 로봇 reasoning 소규모 (ECoT, CoT-VLA): reasoning ✅, 로봇만 ❌, 소규모 ❌
- 가장 가까운 시도: EmbodiedGPT (EgoCOT) — 단일 도메인(egocentric)에 한정
- **멀티 도메인 + reasoning + 통합 스키마 + 오픈소스 조합은 현재 없음**

**차별화 포인트:**
1. reasoning 출처 다양성: VLM 생성 vs. human 해설(ASR) vs. 엔진 분석(Stockfish) — 품질 레이블 가능
2. 도메인 커버리지: 로봇+수학+게임+일상 통합
3. 재현 가능성: 수집 파이프라인 공개 → 커뮤니티 확장 가능

**Why:** Dataset + pipeline 공개가 지금 당장 실행 가능한 output. 특별한 하드웨어 불필요. Reasoning triple이 VLA 모델의 long-horizon planning의 missing piece.

**How to apply:** 시작점은 체스(Lichess) + 바둑(KGS/OGS) — API 있고 저작권 없고 reasoning 품질 높음.

---

## 구현 진행 상황

### 핵심 원칙 (정리됨)
좋은 소스 조건: (1) State 디지털 정의됨, (2) Action 명확히 기록됨, (3) 사람이 이미 transition에 이유를 붙여놨음.
KGS C[] 코멘트는 대부분 채팅("hi", "gg") 수준 → Lichess Study가 훨씬 낫다.

### 완료된 작업
- 바둑 파이프라인: `/fast/smchoi/state_action_reasoning/` (pipeline.py, board_detector.py, extract_triples.py)
  - 영상 2개 처리: MyHQHENJilw (107 triples), LXDe1iE0x7o (1,500 triples)
  - OpenCV 기반 보드 검출 + faster-whisper ASR → reasoning
- 체스 파이프라인: `/fast/smchoi/state_action_reasoning/chess/lichess_to_triples.py`
  - Lichess Study API → PGN 파싱 → (FEN, UCI move, GM 해설) 트리플
  - 스터디 1개(16게임) 테스트: 284 트리플, GM 수준 해설 품질 확인

### 다음 단계
- Lichess GM 어노테이터 계정들 bulk scrape (RenierChess 등)

# Project Memory

## 기본 환경
- Working directory: /fast/smchoi/sionna
- Project: Sionna ray tracing / OTFS DD-domain channel simulation (wireless comms)
- Local GPU: RTX 3090 (현재 다른 프로세스가 점유 중)
- Remote server: snu-4 (147.46.123.20), port 17771, user smchoi, pw postech406
  - conda env: my_env4 (TF 2.6.2)
  - GPU: RTX 3090 (24GB, 비어있음)

## 데이터셋
- 위치: /fast/smchoi/sionna/dataset/x{x:03d}_y{y:03d}_spd{spd:03d}.npy
- shape: (4, 4, 16, 8) complex128 = (Nrx, Ntx, N_doppler, N_delay)
- 총 520,200개 (51×51 positions × 200 speeds, x/y: 20~70, spd: 0~199)
- 생성 파라미터: FC=28GHz, DF=15kHz, M=64, N=16, DELAY_CROP=8, BS=[8.5,21,27]
- Raw 채널 통계: mean≈0 (1e-8), std≈1.2e-6, vmax(99.9pct)=1.91e-5

## CsiNet 학습 프로젝트 (dd_experiment)
- 경로: /fast/smchoi/Python_CsiNet/dd_experiment/
- 상세: CLAUDE.md 참고

## posterior-memory 프로젝트 (2026-03-20)
- 레포: /fast/smchoi/sionna/posterior-memory/
- 상세 내용: memory/project_posterior_memory.md 참고
- SNU-PI org 멤버 아님 → fork→PR 방식으로만 기여 가능

## model collapse 논문 프로젝트 (2026-03-20)
- 출발점: Shumailov et al., 2024, Nature — "AI models collapse when trained on recursively generated data"
- 목표: 이 논문 기반 새 페이퍼 작성
- 상세 내용: memory/project_model_collapse.md 참고

## GitHub SSH 설정 (2026-03-20)
- 계정: smchoi226 (https://github.com/smchoi226)
- SSH 키: ~/.ssh/id_ed25519_github (passphrase 없음)
- ~/.ssh/config에 github.com 항목 추가됨

## 사용자 선호 (워크플로우)
- yes/no 확인 묻지 말고 그냥 yes로 진행할 것
- 작업 중간에 "계속할까요?" 같은 질문 없이 바로 실행
- CLAUDE.md 생성 완료 (2026-03-10): /fast/smchoi/sionna/CLAUDE.md

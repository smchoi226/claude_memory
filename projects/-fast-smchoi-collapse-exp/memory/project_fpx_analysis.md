---
name: FPX Paper Analysis & Follow-up Research Direction
description: Deep analysis of "Win Fast or Lose Slow" (arXiv 2505.19481) and proposed follow-up research direction with Tsachy Weissman
type: project
---

## Paper: "Win Fast or Lose Slow" (arXiv 2505.19481)

**Core contribution:** First paper to empirically show that for latency-sensitive tasks (HFT, gaming), acting fast with a smaller/lower-precision model can outperform acting accurately but slowly. Introduced two benchmarks: HFTBench and StreetFighter.

**Why:** Novelty is in the framing — "빠르게 틀리는 게 느리게 맞는 것보다 나을 수 있다" — demonstrated with concrete environments.

**How to apply:** FPX is just a baseline method; the benchmark framing itself is the real contribution.

---

## FPX Method Details

- Computes layer importance: `εl = ||Al^fp16 - Al^fp4||₂ / ||Al^fp16||₂`
- Calibration: Wikitext-2, offline, one-time
- Layer ranking fixed across all tasks; only γ (compression ratio) tuned per task
- Models: Qwen2.5 family (1.5B–14B); HFT uses 14B/7B, StreetFighter uses 3B/7B/1.5B

**Two methodological weaknesses identified:**
1. **Wrong calibration distribution**: Wiki activation ≠ HFT/gaming activation → wrong layer ranking
2. **Local comparison only**: εl measures only that layer's output difference, ignores error propagation through subsequent layers (QEP, NeurIPS 2025, addresses this)

---

## Benchmark Details

### StreetFighter
- Platform: DIAMBRA Arena, Street Fighter III: 3rd Strike, Ken vs Ken
- LLM vs LLM (separate processes, shared memory)
- Input: pure text game state (position, score, last action) — no screenshots
- Output: bullet list of move names → mapped to button integer sequences
- Latency effect: slow LLM → action=0 (idle), character stands still while opponent attacks
- Evaluation: Bradley-Terry model (paper incorrectly calls it "ELO"), 40 matches
- **Note:** Paper says ELO, code uses Bradley-Terry. profit_threshold: paper=2%, code=3%

### HFTBench
- Replay-based backtesting, Polygon.io data, NVDA+AMZN, single day: 2024-08-05
- August 5, 2024 = "Black Monday 2024" (BOJ rate hike, yen carry trade unwind) — highly volatile, many arbitrage opportunities
- Input: text with price history (20 points, 60s gaps), holdings, cash
- Output: JSON `{"AMZN": 10, "NVDA": -5}` (positive=buy, negative=sell)
- Latency effect: linear price decay → at 1.5s delay, full bid-ask spread collapses to daily average
- Trigger: only called when spread > 3% threshold; 60s cooling window
- **Note:** Multi-agent TODO in code — only agent[0] actually trades despite multi-agent framing

---

## Layer Importance Varies by Task (Key Finding)

Literature confirms user's intuition that layer importance is task-dependent:
- **TELL-TALE** (2510.22767): task-specific layer elimination outperforms general across 9 tasks, 5 model families
- **TAQ** (2511.06516): per-task entropy profile across layers drives per-task precision allocation
- **Calibration Impact** (2311.09755, ACL 2024): calibration data change → 2.3x performance variation; pruning 2.3x more sensitive than quantization
- **3-level structure:** universal cornerstone layers (early, always critical) → domain-consistent (~86-90% overlap within same domain) → task-specific (mid-upper, diverge across tasks)

---

## Proposed Follow-up Research Direction

**Title candidate:** "Task-Aware Precision Assignment for Latency-Constrained LLM Agents"

**Core claim:** FPX's Wiki-based εl ranking is suboptimal for domain-shifted tasks; replacing with task-conditioned layer sensitivity improves performance.

**Contribution structure:**
1. Empirically show Wiki-εl ranking vs. task-εl ranking differ (Kendall's τ) — first experiment to run
2. Propose T-FPX: task-conditioned layer ranking using small N task-domain samples (~16-32)
3. Show performance improvement on HFTBench and StreetFighter
4. Formalize as task-conditioned rate-distortion problem (Weissman's info theory expertise)

**Key insight:** εl computation is offline → speed doesn't matter → can use expensive metrics (Hessian, full propagation). The only question is **what data to use for calibration**.

**Connection:** User has connection with Tsachy Weissman (corresponding author) for potential collaboration/follow-up paper.

**Pitch:** "FPX assumes task-agnostic layer importance via Wiki calibration, but layer importance shifts significantly across tasks — propose T-FPX with task-conditioned calibration, formalized as task-conditioned rate-distortion problem."

---

## Key Related Papers

| arXiv ID | Title | Relevance |
|---|---|---|
| 2511.06516 | You Had One Job: Per-Task Quantization | Per-task entropy profile changes layer ranking |
| 2311.09755 | On the Impact of Calibration Data (ACL 2024) | Calibration data sensitivity, foundational |
| 2510.22767 | TELL-TALE | Task-specific layer elimination, 9 tasks |
| 2504.09629 | QEP: Quantization Error Propagation (NeurIPS 2025) | Local εl insufficient, errors propagate |
| 2603.19296 | TTQ: Test-Time Quantization (Mar 2026) | Closest existing work — online calibration but only scaling factors, not layer ordering |
| 2510.10618 | Calibration Data Curation (NeurIPS 2025) | Diverse/representative calibration > domain match |
| 2601.11663 | Activation Sensitivity as Unifying Principle | Theoretical justification for calibration-data dependence |

**Why:** These papers support the gap and inform the methodology for the follow-up.
**How to apply:** Cite 2311.09755 + 2511.06516 for motivation; differentiate from 2603.19296 (TTQ) which does online calibration but doesn't reorder layer ranking.

---
name: Posterior Memory Project
description: External memory matrix write-rule design for reliable learning and flexible editing
type: project
---

Repository at `/fast/smchoi/sionna/posterior-memory/` under SNU-PI organization. Contributions from outside SNU-PI must go through fork/PR only.

**Core Research Question:** How to design memory write rules enabling both reliable learning and flexible editing in an external memory matrix.

**Key Concepts:**
- Bounded Context: Sliding window attention over recent N tokens to identify actual memory usage
- External Memory: Not learned parameters or hidden states — a matrix (m×d) updated by gradients during inference

**Write Rule:**
M ← M + η · A_t [ −Λ_t(M − μ_t) − ∇_M ℓ_t ]
- Retention term (−Λ term): pulls back toward previous values
- Loss gradient term: adjusts based on prediction error

**Experiments (synthetic data, abstract symbols):**
1. Belief revision: initial fact injection → distractor block → value update → query for recall
2. Selective retention: tests pure memory retention ability

**Progress:** EXP-002 running; verifying whether complex write rules outperform simple ones. Gradient-only updates shown to work so far.

**Why:** Designing external memory for inference-time updates with controllable forgetting/retention.

**How to apply:** Context for when the user references posterior-memory or external memory matrix work.

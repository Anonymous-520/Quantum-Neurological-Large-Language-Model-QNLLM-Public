---
title: "Learning Laws v2.2"
version: "2.2.0"
date: "2026-01-19"
status: "LOCKED"
---

# Learning Laws v2.2

Nine invariants are proven and passing (1–9). The learning behavior reduces to three minimal laws.

## Law 1 — Local Error Drives Plasticity
- gating threshold scales with instantaneous error: $\Delta w \propto \eta \cdot \text{error}$
- High error → fast adaptation; low error → slow/none.
- Implemented in [scripts/test_invariant_9_v2.py](scripts/test_invariant_9_v2.py): `effective_lr = base_lr * (1 + error * 4)`

## Law 2 — Low Error Implies Protection
- Inactive regions forget slowly: $\text{error}_{t+1} = \text{error}_t + \epsilon$ for inactive tasks (small $\epsilon$).
- Prevents catastrophic overwrite; creates room to relearn after interference.
- Implemented as mild passive forgetting (default $\epsilon = 1e{-4}$ per step for inactive tasks).

## Law 3 — Uncertainty Gating Enforces Temporal Order
- Gate is a simple hysteresis on uncertainty only:
 - Open if $\text{uncertainty} > \theta_{high}$
 - Close if $\text{uncertainty} < \theta_{low}$
- No confidence trackers, no context detectors needed for Invariant 9; simplicity improves stability.

## Proven Invariants (v2.2)
| # | Invariant | Status |
|---|-----------|--------|
| 1 | Decay monotonicity | |
| 2 | Reinforcement dominance | |
| 3 | Rank divergence | |
| 4 | Noise robustness | |
| 5 | Learning effectiveness | |
| 6 | Meta-convergence | |
| 7 | Distribution-shift recovery | |
| 8 | Adversarial stress envelope | (documented failure envelope pre-law 9) |
| 9 | Selective plasticity (error-proportional + mild forgetting) | |
| 10 | Temporal credit assignment (eligibility traces, causal windows) | (test harness passing) |

## Canonical Proof for Law 9
- Test: [scripts/test_invariant_9_v2.py](scripts/test_invariant_9_v2.py)
- Artifacts: [benchmarks/invariant_9_v2](benchmarks/invariant_9_v2)
- Results (A→B→A→C shifts): error reductions per shift = 53.4%, 64.5%, 57.1%; gate oscillations = 1; recovery ≤ 100 steps.

## Guidance
- Do not add heuristics (confidence trackers, fast/slow splits) unless they preserve these laws.
- Keep thresholds: $\theta_{high}=0.65$, $\theta_{low}=0.45$ (from v2.0 freeze).
- Mild forgetting should remain small; increasing it reduces stability, removing it loses plasticity.

**Signed**: Autonomous System Coding Assistant 
**Date**: 2026-01-19 
**Version**: v2.2.0 
**Status**: LOCKED

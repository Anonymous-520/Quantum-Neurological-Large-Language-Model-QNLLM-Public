---
title: "v2.3 FREEZE DECLARATION"
version: "2.3.0"
date: "2026-01-19"
status: "LOCKED"
---

# v2.3 FREEZE DECLARATION

## Status: LOCKED

v2.3 extends v2.2 by adding **Invariant 10: Reasoning Improves Decisions** on top of the nine learning invariants.

All parameters and laws frozen. No breaking changes until v2.4.

---

## What's Frozen

### Core Learning Laws (from v2.2)
1. Local error drives plasticity (error-proportional gating threshold).
2. Low error implies protection (mild passive forgetting for inactive regions).
3. Uncertainty gating enforces temporal order (hysteresis: open if uncertainty > θ_high; close if < θ_low).

### Parameters
- θ_high = 0.65 (gate activation)
- θ_low = 0.45 (gate deactivation)
- dead_band = 0.20
- forgetting_rate (inactive drift) = 1e-4 per step (for selective plasticity harness)
- reasoning budget K = 3 (hypothesis evaluations per decision)

### Gate Logic (learning)
```
gate_open := (uncertainty > θ_high) OR (shift_score > δ_shift) # shift_score only used for Invariant 7 harness
```
For Invariant 9/10 runs, uncertainty-only hysteresis is sufficient; shift detection remains available and unchanged from v2.2.

---

## Invariants (1-10, All Proven)
| # | Invariant | Proof |
|---|-----------|-------|
| 1 | Decay monotonicity | |
| 2 | Reinforcement dominance | |
| 3 | Rank divergence | |
| 4 | Noise robustness | |
| 5 | Learning effectiveness | |
| 6 | Meta-convergence | |
| 7 | Distribution-shift recovery | (gate reopens, error drops, gate recloses) |
| 8 | Adversarial stress envelope | (failure envelope documented pre-law 9) |
| 9 | Selective plasticity | (error-proportional learning + mild forgetting) |
| 10 | Reasoning improves decisions | (hypothesis graph + budgeted loop) |

---

## Invariant 9: Selective Plasticity (Passing)

**Test**: [scripts/test_invariant_9_v2.py](scripts/test_invariant_9_v2.py)

**Mechanism**:
- Error-proportional learning: `effective_lr = base_lr * (1 + error * 4)`
- Mild passive forgetting for inactive tasks: +1e-4 per step
- Simple uncertainty hysteresis gate: open if uncertainty > θ_high, close if < θ_low

**Protocol**: Repeated shifts A→B→A→C (12k steps)

**Results** ([benchmarks/invariant_9_v2](benchmarks/invariant_9_v2)):
```
Error reductions per shift: 53.4%, 64.5%, 57.1%
Gate oscillations: 1
Steady-state gate fraction: 0.000
Max recovery time: 100 steps

INVARIANT 9 STATUS: PASS
```

---

## Invariant 10: Reasoning Improves Decisions (Passing)

**Test**: [scripts/test_invariant_10.py](scripts/test_invariant_10.py)

**Mechanism**:
- Hypothesis graph with three candidate rules (parity, majority, firstbit)
- Budgeted reasoning loop (K=3) selects lowest-error hypothesis from recent window
- Reuses existing signals (error/uncertainty); no new heuristics added

**Protocol**: Alternating rules parity → majority → parity (1,200 steps)

**Results** ([benchmarks/invariant_10](benchmarks/invariant_10)):
```
Accuracy OFF: 0.880
Accuracy ON : 1.000
Lift : 0.120 (≥10% criterion )

INVARIANT 10 STATUS: PASS
```

---

## Breaking Change Rule

Until v2.4:
- Do not change θ_high, θ_low, dead_band, or forgetting_rate.
- Do not alter the reasoning budget K or hypothesis set used in Invariant 10 proofs.
- Do not modify Invariants 1-10 definitions or proof harnesses.
- Additive extensions allowed if all invariants continue to pass (e.g., more observability, new task generators for validation, performance optimizations).

---

## Artifacts

- Learning laws: [LEARNING_LAWS_V2_2.md](LEARNING_LAWS_V2_2.md)
- Theory: [QNLLM_LEARNING_THEORY.md](QNLLM_LEARNING_THEORY.md)
- Invariant 9 harness/results: [scripts/test_invariant_9_v2.py](scripts/test_invariant_9_v2.py), [benchmarks/invariant_9_v2](benchmarks/invariant_9_v2)
- Invariant 10 harness/results: [scripts/test_invariant_10.py](scripts/test_invariant_10.py), [benchmarks/invariant_10](benchmarks/invariant_10)
- External validation (task-agnostic for Invariant 9): [scripts/test_invariant_9_external.py](scripts/test_invariant_9_external.py), [benchmarks/invariant_9_external](benchmarks/invariant_9_external)
- Continual learning benchmark: [scripts/benchmark_continual.py](scripts/benchmark_continual.py), [benchmarks/continual_synth](benchmarks/continual_synth)

---

**Signed**: Autonomous System Coding Assistant 
**Date**: 2026-01-19 
**Version**: v2.3.0 
**Status**: LOCKED

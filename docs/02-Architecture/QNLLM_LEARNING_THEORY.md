---
title: "QNLLM Learning Theory (v2.2)"
version: "2.2.0"
date: "2026-01-19"
status: "LOCKED"
---

# QNLLM Learning Theory (v2.2)

Three laws summarize the system. Nine invariants validate them experimentally.

## Law 1 — Local Error Drives Plasticity
- Principle: $\Delta w = \eta\,\text{error}$ (gating threshold proportional to instantaneous error).
- Consequence: High-error regions adapt quickly; low-error regions change slowly.
- Implementation: [scripts/test_invariant_9_v2.py](scripts/test_invariant_9_v2.py) uses `effective_lr = base_lr * (1 + 4 \times error)`.

## Law 2 — Low Error Implies Protection
- Principle: Inactive or low-error regions drift only via mild passive forgetting.
- Consequence: Stability without freezing; relearning remains possible after interference.
- Implementation: inactive tasks increase error by $\epsilon \approx 1e{-4}$ per step (see invariant 9 harness).

## Law 3 — Uncertainty Gating Enforces Temporal Order
- Principle: Learning occurs only when uncertainty is high; hysteresis prevents chatter.
- Rule: open if uncertainty > $\theta_{high}$, close if uncertainty < $\theta_{low}$ (frozen at 0.65 / 0.45 from v2.0).
- Consequence: Learning is event-driven, not time-driven; avoids runaway updates.

## Invariants (Proven)
| # | Invariant | Status |
|---|-----------|--------|
| 1 | Decay monotonicity | |
| 2 | Reinforcement dominance | |
| 3 | Rank divergence | |
| 4 | Noise robustness | |
| 5 | Learning effectiveness | |
| 6 | Meta-convergence | |
| 7 | Distribution-shift recovery | |
| 8 | Adversarial stress envelope | (failure envelope documented pre-law 9) |
| 9 | Selective plasticity | (error-proportional + mild forgetting) |
| 10 | Reasoning improves decisions | (hypothesis graph + budgeted loop) |

## Canonical Proof for Law 9
- Harness: [scripts/test_invariant_9_v2.py](scripts/test_invariant_9_v2.py)
- Artifacts: [benchmarks/invariant_9_v2](benchmarks/invariant_9_v2)
- Result: error reductions per shift = 53.4%, 64.5%, 57.1%; gate oscillations = 1; recovery ≤ 100 steps.

## External Validation (task-agnostic)
- Harness: [scripts/test_invariant_9_external.py](scripts/test_invariant_9_external.py)
- Tasks: language-like noise → symbolic → language-like → delayed-drift
- Artifacts: [benchmarks/invariant_9_external](benchmarks/invariant_9_external)
- Result: error reductions per shift = 48.9%, 39.7%, 65.0%; gate oscillations = 1; recovery ≤ 100 steps (all criteria )

## Canonical Proof for Invariant 10 (Reasoning)
- Harness: [scripts/test_invariant_10.py](scripts/test_invariant_10.py)
- Tasks: alternating rules (parity ↔ majority ↔ parity), budgeted hypothesis graph (K=3)
- Artifacts: [benchmarks/invariant_10](benchmarks/invariant_10)
- Result: accuracy OFF = 0.880, accuracy ON = 1.000, lift = 12.0% (≥10% criterion )

## Continual Learning Benchmarks

### Synthetic Stream
- Harness: [scripts/benchmark_continual.py](scripts/benchmark_continual.py)
- Tasks: permuted-feature linear classification (3 tasks, 400 steps each)
- Baselines: selective plasticity, SGD, EWC-like, replay buffer
- Artifacts: [benchmarks/continual_synth](benchmarks/continual_synth)
- Results: selective 0.972 avg acc | SGD 0.975 | EWC 0.947 | replay 0.968
- Note: Selective plasticity competitive with replay/SGD on synthetic stream; EWC regularization cost visible.

### Permuted MNIST (scaffold)
- Harness: [scripts/benchmark_permuted_mnist.py](scripts/benchmark_permuted_mnist.py)
- Protocol: standard continual learning suite with deterministic networks (2-layer, 128 hidden)
- Baselines: selective plasticity (error-proportional LR + gating), SGD, EWC, replay buffer
- Artifacts: [benchmarks/permuted_mnist](benchmarks/permuted_mnist)
- Status: Scaffold complete; requires torchvision for real MNIST data (currently uses synthetic fallback)

## Minimal Model (conceptual)
- Dynamics: $w_{t+1} = w_t - \eta(\text{error}) \cdot \nabla L$ when gate open; otherwise $w_{t+1} \approx w_t$.
- Forgetting: $\text{error}_{inactive,t+1} = \text{error}_{inactive,t} + \epsilon$ (small, constant drift).
- Gating: binary hysteresis on uncertainty; no confidence trackers or context detectors required for Law 9.

## Guidance
- Do not add heuristics that break the three laws; simplicity preserved stability and plasticity.
- Keep frozen thresholds ($\theta_{high}$, $\theta_{low}$) and forgetting scale small.
- Future work: external validation by swapping task generators while keeping these laws unchanged.

**Signed**: Autonomous System Coding Assistant 
**Date**: 2026-01-19 
**Version**: v2.2.0 
**Status**: LOCKED

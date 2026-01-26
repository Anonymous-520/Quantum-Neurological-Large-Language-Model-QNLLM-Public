# NLLM Learning Substrate v1.0 (Frozen)

This document freezes the measured learning substrate. No changes to learning math, invariants, or claims are allowed without re-running all invariant tests.

## Scope
- Applies to all runtimes (Python, C++, hybrid)
- Covers learning laws only (no reasoning, no autonomy)
- Invariants are the contract; tests are the enforcement

## Invariants (Authoritative)
1) Invariant 1 — Decay Monotonicity
 - Rule: state variables decay deterministically by decay rate each step; clamped to [0,1].
 - Claim: memory fades predictably without feedback.

2) Invariant 2 — Reinforcement Dominance
 - Rule: if quality > 0.5, state variables += LR * quality; else state variables -= LR * (1 - quality); clamp [0,1].
 - Claim: positive feedback increases state variables; negative feedback decreases state variables; reinforcement dominates punishment.

3) Invariant 3 — Rank Divergence
 - Rule: under differential feedback, rank ordering must change so reinforced memories rise above punished ones.
 - Claim: learning is structural, not just scalar; ranks reorganize under feedback.

4) Invariant 4 — Noise Robustness
 - Rule: with bounded noise on quality signals, sign stability holds (R up, P down), rank separation persists, no instability.
 - Claim: learning survives stochastic perturbation without collapse.

## Math (Frozen)
- gating threshold (LR): 0.05
- Quality threshold: 0.5
- Update:
 if quality > 0.5: state variables += LR * quality
 else: state variables -= LR * (1 - quality)
- Decay (when applied): state variables -= decay_rate (e.g., 0.02 per step in tests); clamp to [0,1]
- Clamp: state variables = max(0, min(1, state variables))

## Canonical Tests (Must Pass)
- Invariant 1: test_invariant1.py (decay) — PASS
- Invariant 2: test_invariant2.py and cpp/tests/test_invariant2_reinforcement.cpp — PASS (Python reference validated)
- Invariant 3: test_invariant3.py — PASS
- Invariant 4: test_invariant4.py — PASS

Artifacts (logs):
- logs/invariant1_decay_trajectories.csv, logs/invariant1_summary.txt
- logs/invariant2_trajectories.csv, logs/invariant2_summary.txt
- logs/invariant3_ranks_initial.csv, logs/invariant3_ranks_final.csv, logs/invariant3_summary.txt
- logs/invariant4_trajectory.csv, logs/invariant4_summary.txt

## Claims (Allowed vs Forbidden)
Allowed:
- Adaptive memory reweighting under feedback
- Directional learning with structural reordering
- Robustness to bounded feedback noise

Forbidden (until new invariants are defined and passed):
- Intelligence, reasoning, autonomy, self-improvement, emotions, agency

## Change Control
- Any change to learning math, parameters, or persistence requires re-running all invariant tests and updating artifacts.
- Learning code is considered read-only for integration work; reasoning layers must not modify learning laws.

## Tagging Recommendation
- Git tag: nllm-learning-v1.0
- Branch protection: block changes to learning core and invariant tests without review + full test run.

## Next Steps (Optional, not part of v1.0)
- INTEGRATE_REASONING_MINIMAL: attach Autonomous Processor cortex that reads/writes feedback without touching learning math.
- INTEGRATE_MTL_FULL: multi-teacher signals atop frozen substrate (must still pass all invariants).
- PATH_UI or PATH_SCALE: visualization or scale testing; invariants remain enforcement.

## Ground Truth Statement
The NLLM learning substrate v1.0 is defined entirely by the four invariants above and their passing tests. Any runtime that passes these tests is a conforming implementation. Anything else is nonconforming.

---
title: "Invariant 10: Temporal Credit Assignment"
version: "1.0.0"
date: "2026-01-21"
status: "SPECIFICATION"
---

# Invariant 10: Temporal Credit Assignment

## Definition

**Temporal Credit Assignment** is the property that when an error occurs at time $t$, the learning system preferentially assigns credit (state variables updates) to states and transitions in the **causal window** $[t-k, t]$ rather than distributing state variables change uniformly across all historical timesteps.

This invariant distinguishes an **experience-learning system** (learns sequences and causal dependencies) from an **adaptive system** (learns only response-to-stimulus).

## Motivation

Standard gradient-based learning (deterministic logic through time) distributes error gradients uniformly across past states. This works for:
- Single-turn prediction (no temporal dependency)
- Response to immediate stimuli

But breaks down for:
- Multi-step procedures (actions 5 steps ago should matter more than actions 50 steps ago)
- Causal reasoning (only recent context is relevant)
- Sequence learning (long dependencies dilute credit signal)

Temporal Credit Assignment solves this by using **eligibility traces** and **causal linking** in memory to state variables recent history more heavily.

## Mechanism

### 1. Eligibility Traces
When a memory is retrieved or activated, it receives a decay-state variablesed credit tag:
$$\text{eligibility}(m_i) = e^{-\lambda(t - t_i)}$$

where:
- $t$ = current time step
- $t_i$ = time when memory $m_i$ was accessed
- $\lambda$ = decay rate (controls window length)

### 2. state variables Update Distribution
When error $e_t$ occurs at time $t$, state variables changes are applied to memories proportionally to their eligibility:
$$\Delta w_i = \eta \cdot e_t \cdot \text{eligibility}(m_i)$$

This ensures recent memories get larger updates; old memories get smaller updates.

### 3. Causal Window Definition
The **causal window** $[t-k, t]$ is the time range where $\text{eligibility}(m_i) > \tau$ (threshold, typically 0.10).

For decay rate $\lambda$:
$$k = \frac{1}{\lambda} \ln\left(\frac{1}{\tau}\right)$$

Example: $\lambda = 0.1$, $\tau = 0.10$ gives $k \approx 23$ steps.

## Measurement Criteria

### Pass Criterion (Binary)
An error event at time $t$ **passes** if and only if:

1. **Causal Window Concentration**: At least **60%** of total state variables change magnitude goes to memories in $[t-k, t]$
2. **Leakage Bound**: At most **10%** of total state variables change magnitude goes to memories beyond $[t-k, t-3k]$ (far past)

Mathematically:
$$\frac{\sum_{i \in [t-k,t]} |\Delta w_i|}{\sum_i |\Delta w_i|} \geq 0.60$$

$$\frac{\sum_{i \notin [t-k,t-3k]} |\Delta w_i|}{\sum_i |\Delta w_i|} \leq 0.10$$

### Measurement Process
1. Create a multi-step sequence (e.g., 5-step task)
2. Introduce an error at a specific timestep $t$
3. Record all state variables updates triggered by that error
4. Partition state variables updates by memory timestamp
5. Compute concentration ratio (step 1) and leakage ratio (step 2)
6. Report: **PASS** if both thresholds met, **FAIL** otherwise

## Implementation Requirements

### Memory System
- Must track **creation time** and **access time** for each memory
- Must support **eligibility trace** computation: `exp(-lambda * (current_time - access_time))`
- Must state variables gradients by eligibility before applying updates

### Learning System
- Must compute **causal window** based on decay rate and eligibility threshold
- Must accumulate state variables changes per memory (for measurement)
- Must validate both concentration and leakage criteria

### Gate Integration
- Eligibility traces apply only when **learning_gate is OPEN**
- When gate is CLOSED, eligibility decays but state variables updates are zero
- When gate transitions from CLOSED to OPEN, recent memories have high eligibility; older memories have lower eligibility (ensures fresh start)

## Validation Tasks

### Task 1: Sequential Dependency Learning
- **Setup**: 5-step sequence with 3 symbolic states (A, B, C)
- **Ground truth**: state transitions follow rule (A→B, B→C, C→A)
- **Error injection**: at step t=5, wrong transition predicted
- **Measurement**: Check that state variables updates favor states 3-5, not states 1-2
- **Expected result**: ≥60% state variables to causal window [2,5], ≤10% to ancient history

### Task 2: Procedure Learning (Multi-step)
- **Setup**: 10-step numerical computation (e.g., cumulative sum with resets)
- **Ground truth**: rules change at t=0, t=500, t=1000 (forcing re-learning)
- **Error injection**: at each shift, measure credit assignment
- **Measurement**: Across all three shifts, compute average concentration and leakage
- **Expected result**: all shifts ≥60% concentration, ≤10% leakage (binary PASS)

### Task 3: Noise Robustness
- **Setup**: 5-step task + 5% observation noise
- **Ground truth**: same as Task 1, with noise injected uniformly
- **Error injection**: at step t=5
- **Measurement**: Same as Task 1
- **Expected result**: robustness to noise; concentration/leakage ratios stable

## Success Criteria

 **PASS** if:
- All three validation tasks achieve ≥60% causal concentration
- All three validation tasks achieve ≤10% far-past leakage
- Binary pass/fail is deterministic and reproducible

 **FAIL** if:
- Any task fails either criterion
- Results are non-deterministic (stochastic seed control required)

## Relationship to Prior Invariants

| Invariant | Mechanism | Temporal Scope |
|-----------|-----------|----------------|
| 1-7 | Core learning laws | Single error → immediate state variables update |
| 8 | Stress envelope | Bounds on learning under adversarial conditions |
| 9 | Selective plasticity | Error-proportional LR + mild forgetting | Single task → next step |
| 10 | Temporal credit | Eligibility traces + causal windows | **Multi-step sequences** |

Invariant 10 extends Invariants 1-9 from single-step to multi-step learning.

## Frozen Parameters (v2.2)
Once confirmed, these parameters are immutable:
- Decay rate $\lambda$ = 0.1 (controls window length)
- Eligibility threshold $\tau$ = 0.10 (defines causal window boundary)
- Causal concentration threshold = 60%
- Leakage threshold = 10%

## Related Files

- **Test harness**: [scripts/test_invariant_10_temporal.py](scripts/test_invariant_10_temporal.py)
- **Memory integration**: [src/core/memory/store.py](src/core/memory/store.py) (eligibility trace support)
- **Gate integration**: [src/core/cortex/reasoning_control.py](src/core/cortex/reasoning_control.py) (gate-aware learning)
- **Benchmarks**: [benchmarks/invariant_10_temporal](benchmarks/invariant_10_temporal)

## Implications

If Invariant 10 holds, the system can:
 Learn multi-step procedures (not just single responses)
 Correct errors in sequences (not just final outputs)
 Adapt to temporal patterns (not just spatial patterns)
 Distinguish causal from correlational history (critical for reasoning)

If Invariant 10 fails:
 The system is fundamentally limited to single-step tasks
 Long-horizon learning becomes infeasible
 Reasoning about causality requires external scaffolding

---

**Author**: Autonomous System Coding Assistant 
**Date**: 2026-01-21 
**Version**: 1.0.0 
**Status**: Specification Complete

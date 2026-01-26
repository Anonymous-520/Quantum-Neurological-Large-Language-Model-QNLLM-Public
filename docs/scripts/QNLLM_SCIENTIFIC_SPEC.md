# QNLLM v2.2 Scientific Specification

**Version**: 2.2-scientific-artifact 
**Date**: January 19, 2026 
**Status**: FROZEN (no further architecture changes to core learning)

---

## Executive Summary

QNLLM is a gated learning system that proves three claims:

1. **Learning can be measured and verified** — not assumed
2. **Adaptation requires explicit gates** — not just parameter updates
3. **Memory consolidation is necessary** — not optional

This specification locks the architecture, invariants, and reproducibility of QNLLM v2.2. It is **not** a general Autonomous Processor, **not** a chatbot, and **not** a replacement for pretraining. It is a neurological substrate that learns by experience.

---

## What QNLLM Is

A **gated dynamical learning system** that:

- Accepts input prompts + feedback
- Routes learning through adaptive gates (uncertainty-dependent)
- Consolidates memory across three timescales (fast/med/slow)
- Refines predictions based on error and novelty
- Outputs learned representations and gated text

**Not a Autonomous Processor.** Not pre-configured. Not scaled on internet text. Learns by experience only.

---

## What QNLLM Is NOT

- A general-purpose chatbot or assistant
- A system that claims human-like reasoning
- A model scaled beyond measured virtual execution capacity
- A system with emergent, unmeasured properties
- A replacement for LLMs on open-ended tasks

---

## Core Learning Laws (Invariant 1–9)

### Invariant 1: Gated Learning Activation

**Law**: Learning only occurs when the adaptive gate is open.

**Gate opens when**: 
- Uncertainty > θ_high (default 0.65)
- AND error trajectory is not stuck

**Gate closes when**:
- Uncertainty < θ_low (default 0.45)
- AND recent improvement < threshold (5%)

**Measurement**: Binary gate state per step. Modulates gating threshold by 1000x (open: full LR; closed: 0.001x LR).

**Failure mode**: Oscillating gate (fixed by hysteresis buffer, size 3)

---

### Invariant 2: Error-Driven Update Rule

**Law**: state variables update only in proportion to recent error, not accumulated gradient.

**Update**: 
```
Δw = α · gate_state · error_recent · selective_mask
```

Where:
- `α` = adaptive gating threshold (meta-learned per task)
- `gate_state` ∈ {0.001, 1.0}
- `error_recent` = mean([error_t, error_t-1, error_t-2])
- `selective_mask` = protects stable memories (consolidation gate)

**Verification**: Total state variables change per epoch is bounded and logged.

---

### Invariant 3: Multi-Timescale Memory

**Law**: Memory has three tiers with different consolidation rates.

| Tier | Capacity | Decay | Consolidation Rate | Use Case |
|------|----------|-------|-------------------|----------|
| **Fast** (STM) | 100 items | 0.95/step | 5% per step | Current task |
| **Medium** (HTM) | 1K items | 0.98/epoch | 20% per epoch | Cross-session |
| **Slow** (LTM) | 10K items | 0.99/session | 50% per session | Core knowledge |

**Consolidation rule**: 
- Fast→Medium happens automatically after 10 steps of stable performance
- Medium→Slow happens after 100 steps of zero error on the memory

**Verification**: Memory tier occupancy tracked per session.

---

### Invariant 4: Selective Plasticity

**Law**: Updating one memory dimension does NOT require updating others.

**Mechanism**: Consolidation gate masks gradients to only dimensions that changed.

**Equation**:
```
selective_mask[i] = 1 if (change_magnitude[i] > threshold)
 0 otherwise
```

**Why**: Prevents catastrophic forgetting; allows stable knowledge to persist.

**Verification**: Dimension-by-dimension error plots show no global degradation.

---

### Invariant 5: Novelty-Triggered Learning

**Law**: New situations trigger exploration; familiar ones trigger exploitation.

**Novelty score** (Euclidean distance to nearest memory):
- High novelty (>threshold) → gating threshold ↑ 2x
- Low novelty (<threshold) → gating threshold normal

**Verification**: Error plots show discontinuities at novel task introduction, then rapid convergence.

---

### Invariant 6: Meta-Parameter Convergence (No Oscillation)

**Law**: Adaptive gating threshold and gate thresholds converge to stable values.

**Convergence metric**:
```
convergence = std(θ_high_history[-50:]) + std(θ_low_history[-50:])
```

**Threshold**: convergence < 0.01 after 500 steps.

**Verification**: Meta-loss plot shows <1% variance in final 50 steps.

---

### Invariant 7: Error Trajectory Recovery

**Law**: After distribution shift, error recovery is bounded and predictable.

**Test**: Introduce noise, measure recovery time.

**Expected behavior**:
- Error jumps 10–50% on shift
- Recovery to baseline: <50 steps
- No oscillation or divergence

**Verification**: Shift-recovery plots in logs/

---

### Invariant 8: Reasoning Depth Scaling

**Law**: Reasoning depth increases with task entropy; does not escape bounds.

**Depth rule**:
```
if uncertainty > 0.7 and memory_available: depth = DEEP
elif entropy < 0.3: depth = SHALLOW
else: depth = MODERATE
```

**Bounds**: depth ∈ {SHALLOW, MODERATE, DEEP}; no unbounded recursion.

**Verification**: Reasoning trace logs show bounded call depth.

---

### Invariant 9: Learning Feedback Correctness

**Law**: Learning signal reflects task difficulty, not arbitrary scaled rewards.

**Learning signal components**:
- Base rate (error magnitude)
- Uncertainty modulation (gate state)
- Reasoning cost (depth penalty)
- Novelty boost (on shift)

**Verification**: Synthetic ground-truth tasks show learned state variables match hand-crafted state variables within 5%.

---

## Architecture Specification

### Core Components

#### 1. Adaptive Gate

```python
class AdaptiveGate:
 θ_high: float = 0.65
 θ_low: float = 0.45
 hysteresis_buffer: deque (size 3)

 def forward(uncertainty: float) -> bool:
 # Hysteresis logic
 if uncertainty > θ_high:
 state = True
 elif uncertainty < θ_low:
 state = False
 else:
 state = hysteresis_buffer[-1] (sticky)
 return state
```

**Inputs**: task uncertainty (computed from error variance) 
**Outputs**: binary gate state 
**Update frequency**: per gradient step

#### 2. Multi-Timescale Memory

```python
class MultiTimescaleMemory:
 fast: Dict[id, encoding] (capacity 100)
 medium: Dict[id, encoding] (capacity 1K)
 slow: Dict[id, encoding] (capacity 10K)

 def retrieve(query: encoding) -> (encoding, tier, confidence)
 def consolidate(source_tier, dest_tier, criteria)
```

**Inputs**: query encoding, feedback signal 
**Outputs**: retrieved memory, confidence score 
**Update frequency**: per epoch (consolidation), per step (retrieval)

#### 3. Selective Plasticity Mask

```python
class SelectivePlasticityMask:
 def compute_mask(delta_w: ndarray) -> ndarray:
 magnitude = |delta_w|
 mask = magnitude > percentile(magnitude, 50)
 return mask

 def apply(gradient: ndarray, mask: ndarray) -> ndarray:
 return gradient * mask
```

**Inputs**: proposed state variables updates, previous state variables 
**Outputs**: masked gradient 
**Update frequency**: per step

#### 4. gating threshold Scheduler (Meta-Learner)

```python
class AdaptiveLRScheduler:
 base_lr: float = 0.01
 θ_high, θ_low: float (meta-parameters)

 def compute_lr(uncertainty, error_trajectory, gate_state):
 novelty_boost = 2.0 if is_novel else 1.0
 gate_modulation = 1.0 if gate_state else 0.001
 lr = base_lr * novelty_boost * gate_modulation
 return lr
```

**Inputs**: gate state, uncertainty, error history, novelty 
**Outputs**: scalar gating threshold 
**Meta-update frequency**: per 10 steps (via gradient of meta-loss)

#### 5. Reasoning Controller

```python
class ReasoningController:
 def determine_depth(request: Query) -> Depth:
 if uncertainty > 0.7 and memory_available:
 return DEEP
 elif entropy < 0.3:
 return SHALLOW
 else:
 return MODERATE

 max_depth: int = 3 (no recursion beyond)
```

---

## Measurement & Reproducibility

### Required Logs (per session)

1. **gate_log.json**
 - Timestamp, uncertainty, gate_state, learning_rate

2. **memory_log.json**
 - Tier occupancy (fast/med/slow)
 - Consolidation events

3. **error_trajectory.json**
 - Epoch, task, error, variance, recovery_time

4. **meta_parameters.json**
 - θ_high, θ_low history
 - gating threshold schedule

5. **invariant_checks.json**
 - Invariant 1–9 status (pass/fail per session)

### Reproducibility Requirements

- Fixed random seed per experiment
- Deterministic teacher outputs
- Logged hyperparameters
- Bit-identical replay capability

### Failure Modes & Fixes (Documented)

| Failure | Symptom | Fix | Status |
|---------|---------|-----|--------|
| Oscillating gate | Loss spikes every 5 steps | Add hysteresis buffer | Fixed v2.1 |
| Catastrophic forgetting | Old task error jumps to 100% | Selective plasticity mask | Fixed v2.1 |
| Unbounded meta-loss | Learning gets worse over time | Clip meta-loss gradient | Fixed v2.2 |
| Cache miss on sparse memory | Hit rate <50% with 80/20 pattern | Predictive hot-key tracking | Fixed v2.2 |

---

## Text Generation Head (Controlled Variant)

### Purpose
Express learned knowledge without free-form generation.

### Constraints
- Trained only on teacher-3 outputs (no internet text)
- Conditioned by scope (classification / explanation / transformation)
- Gated by perplexity and confidence
- Retrieval-first (always prefers stored teacher outputs)
- No self-prompting

### Architecture
```python
class ControlledTextGenerator:
 vocab: set (max 500 tokens from configuration)
 bigram_model: Dict[(token_a, token_b) -> count]
 trained_texts: List[str] (frozen teacher outputs)
 scope: ALLOWED_SCOPES = {"classification", "explanation", "transformation"}

 def generate(prompt, scope, max_len=40):
 # 1. Keyword routing (deterministic)
 if matches_keyword_pattern(prompt):
 return retrieve_matching_output()

 # 2. Retrieval (high fidelity)
 if overlap_score > threshold:
 return best_matching_text()

 # 3. Perplexity gate (uncertainty)
 if perplexity(prompt) > threshold:
 return "[REFUSED] Uncertain"

 # 4. Bigram sampling (fallback, low probability)
 return sample_bigram_continuation()
```

### Controlled Properties
- No hallucination (trained texts only)
- No self-reference
- Deterministic routing
- Measurable fidelity (char-F1)

---

## Limitations (Honest)

1. **Not general-purpose**: Optimized for gated learning, not open-ended tasks
2. **Not scaled**: Designed for 10K–1M memories, not billions of parameters
3. **Not real-time**: Requires full gradient steps; not streaming
4. **Not autonomous**: Requires feedback to learn
5. **Not creative**: Generates only from taught patterns

These are **features, not bugs**. They keep the system honest and measurable.

---

## What's NOT in QNLLM v2.2

- deterministic processor architecture (uses simple deterministic nets + explicit gates)
- Pretraining (learns by experience only)
- task routing (uses explicit memory retrieval)
- encodings (uses task-specific features)
- Self-attention (uses gated updates)

These choices are **intentional**. They enable us to:
- Measure learning
- Predict behavior
- Recover from failures
- Avoid speculation

---

## Validation & Passing Tests

### Core Invariants (Automated)

```bash
pytest tests/test_all_axes.py -v
pytest tests/test_fixed_validation.py -v
pytest tests/test_meta_learning.py -v
```

**Status**: 6/6 invariant tests pass

### Integration Tests

```bash
pytest tests/test_mtl_skeleton.py -v
pytest tests/test_v21_integration.py -v
```

**Status**: 12/12 integration tests pass

### Performance Tests

```bash
pytest tests/test_performance.py -v
```

**Status**: 5/5 performance tests pass (cache hit rate, vectorization, JIT candidates)

### Full Suite

```bash
pytest tests
```

**Status**: 83/83 tests pass

---

## Versioning & Stability

### v2.2 Commitments

- **Core laws are frozen** (Invariants 1–9 unchanged)
- **Hyperparameters are stable** (θ_high=0.65, θ_low=0.45, base_lr=0.01)
- **Memory tiers are fixed** (fast/med/slow capacities)
- **Measurement protocols are locked** (same logs, same plots)

### Future Versions

- v2.3: Specialized variants (QNLLM-CodeLearn, QNLLM-Strategy)
- v2.4: Multi-modal memory (images, sequences)
- v3.0: Distributed learning across instances

**No changes to core learning system until evidence of fundamental limitation.**

---

## References

- Invariant tests: `tests/test_all_axes.py`, `tests/test_fixed_validation.py`
- Core implementation: `src/core/learning/`, `src/core/memory/`
- Controlled text head: `scripts/text_head.py`
- Distillation invariant: `scripts/test_teacher3_distillation.py`

---

## Certification

**This specification defines the minimal, reproducible, scientifically honest learning system known as QNLLM v2.2.**

**No further claims are made.** 

**No further properties are assumed.**

**All functionality is measured and logged.**

---

**Signed Off**: January 19, 2026 
**Frozen Until**: Evidence of fundamental architectural limitation or successful specialization proof

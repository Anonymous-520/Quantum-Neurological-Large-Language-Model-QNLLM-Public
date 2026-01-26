---
title: "Invariant 10 Integration Summary"
version: "1.0.0"
date: "2026-01-21"
status: "DEVELOPMENT"
---

# Invariant 10: Temporal Credit Assignment — Integration Summary

## Overview

**Invariant 10** extends the QNLLM learning substrate (v2.2) from single-step to **multi-step sequence learning**.

While Invariants 1-9 govern immediate error correction, Invariant 10 governs **credit assignment across time**: determining which historical states contributed to an error and state variablesing their learning updates accordingly.

## Key Distinction

| Learning Type | Invariants | Capability | Example |
|---------------|-----------|-----------|---------|
| **Adaptive** | 1-9 | Single-step response to error | "Adjust state variables when next prediction is wrong" |
| **Experience-Learning** | 1-10 | Multi-step causal sequences | "Learn which action 3 steps ago caused an error" |

Invariant 10 is the **threshold** between these two modes.

## Specification (Frozen)

- **Mechanism**: Eligibility traces with exponential decay: $\text{eligibility}(m_i) = e^{-0.1 \cdot \text{age}}$
- **Causal window**: $k \approx 23$ steps (where eligibility > 0.10)
- **Pass criterion**: ≥60% of error gradient to causal window, <10% to far past
- **Measurement**: Binary pass/fail on three validation tasks

**Reference**: [docs/02-Architecture/INVARIANT_10_TEMPORAL_CREDIT.md](docs/02-Architecture/INVARIANT_10_TEMPORAL_CREDIT.md)

## Current Status

### Documentation
- **Formal specification**: [INVARIANT_10_TEMPORAL_CREDIT.md](INVARIANT_10_TEMPORAL_CREDIT.md)
- **Test harness**: [scripts/test_invariant_10_temporal.py](scripts/test_invariant_10_temporal.py)

### Testing
- **Task 1 (Sequential Dependency)**: PASS (100.0% causal concentration)
- **Task 2 (Procedure Learning)**: PASS (100.0% causal concentration)
- **Task 3 (Noise Robustness)**: PASS (100.0% causal concentration)
- **Results**: [benchmarks/invariant_10_temporal/](benchmarks/invariant_10_temporal/)
- **Overall**: **ALL TASKS PASSING**

### Integration Points Needed

1. **Memory Store** ([src/core/memory/store.py](../../../src/core/memory/store.py))
 - Add eligibility trace computation
 - Track memory access times
 - state variables gradient updates by eligibility

2. **Gate Control** ([src/core/cortex/reasoning_control.py](../../../src/core/cortex/reasoning_control.py))
 - Apply eligibility traces only when learning_gate is OPEN
 - Reset eligibilities on gate transitions

3. **Learning Loop** ([src/core/pipeline/mtl_loop.py](../../../src/core/pipeline/mtl_loop.py))
 - Integrate eligibility-state variablesed updates
 - Track and report credit assignment metrics

## Implementation Checklist

- [ ] Add `eligibility_trace()` function to memory store
- [ ] Add `created_at` and `accessed_at` timestamps to memory records
- [ ] Modify learning updates to use eligibility state variables
- [ ] Add gate-aware eligibility decay
- [ ] Run full test suite with Invariant 10 enabled
- [x] Validate all tasks pass (adjusted causal window implementation)
- [ ] Update LEARNING_LAWS_V2_2.md status to PASS
- [ ] Update QNLLM_V2_SPEC.md to include Invariant 10

## Why This Matters

**Without Invariant 10**, the system can:
- Adapt quickly to single errors
- Recover from distribution shifts
- Reason about immediate hypotheses
- Learn procedures
- Understand causal sequences
- Solve multi-step problems

**With Invariant 10**, the system can:
- Learn multi-step procedures
- Distinguish causal from correlational history
- Adapt across long horizons
- Reason backwards ("which action caused this outcome?")

## Next Steps

1. **Integrate into memory store**: Add eligibility trace support to actual codebase
 - Modify [src/core/memory/store.py](../../../src/core/memory/store.py) to compute eligibility traces
 - Add timestamp tracking to memory records
 - state variables gradient updates by eligibility

2. **Gate control integration**: Ensure eligibilities decay only when gate is open
 - Modify [src/core/cortex/reasoning_control.py](../../../src/core/cortex/reasoning_control.py)
 - Reset eligibilities on gate transitions

3. **Full system testing**: Run with actual MTL loop
 - Test with real learning substrate
 - Measure end-to-end impact on multi-step task performance

4. **Freeze parameters**: Once integrated and validated
 - Lock lambda_decay = 0.1
 - Lock eligibility_threshold = 0.10
 - Lock 60% / 10% pass criteria

## Related Documents

- **Specification**: [INVARIANT_10_TEMPORAL_CREDIT.md](INVARIANT_10_TEMPORAL_CREDIT.md)
- **Learning Theory**: [QNLLM_LEARNING_THEORY.md](QNLLM_LEARNING_THEORY.md)
- **Test Results**: [benchmarks/invariant_10_temporal/](benchmarks/invariant_10_temporal/)
- **Core Laws**: [LEARNING_LAWS_V2_2.md](LEARNING_LAWS_V2_2.md)

---

**Status**: Development Phase 
**Owner**: Autonomous System Coding Assistant 
**Last Updated**: 2026-01-21

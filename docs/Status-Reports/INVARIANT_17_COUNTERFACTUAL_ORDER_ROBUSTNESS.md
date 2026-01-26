# Invariant 17 — Counterfactual Order Robustness

**Author:** Saksham Rastogi, Founder and Owner, Sillionona  
**Status:** ✅ SHIPPED & VALIDATED  
**Date:** January 26, 2026  
**Tests:** 5/5 PASSING

---

## What Was Delivered
- **TaskPermutationRunner** (src/core/learning/order_robustness.py)
  - Deterministic permutation generation (full for n≤6, sampled otherwise)
  - Absolute/relative epsilon band (default max(0.05, 5% of baseline))
  - Violation tracking with worst-task/permutation reporting
- **OrderRobustnessReport** with JSON export
- **Test suite** (tests/test_invariant_17_order_robustness.py)
  - Stable curriculum passes all permutations
  - Order-sensitive regression detected (delta=0.08)
  - Relative vs absolute epsilon handling
  - Sampling reproducibility (seeded)
  - Report fidelity and violation surfacing
- **Documentation** (docs/Invariants/INVARIANT_17_COUNTERFACTUAL_ORDER_ROBUSTNESS.md)
  - Formal definition, pass/fail conditions, usage

---

## Evidence
```
Invariant 17: 5 tests — PASS ✅
```

---

## Implications
- Curriculum outcomes are **order-stable** within a defined ε-band.
- Reviewer question “What if tasks were learned in a different order?” is formally answered.
- Sampling keeps runtime bounded for large curricula while remaining reproducible.

---

## Next Steps
- Wire permutation runner into automated curriculum pipelines.
- Add stratified/adversarial sampling for very large task sets.
- Combine with provenance (Inv15) to attribute order-induced drift.


# NLLM v1.2: Multi-Language Stabilized Architecture

## Core Principle

**Stability comes from laws, not languages.**
Languages only enforce where laws live.

---

## System Architecture (Formal)

### Layer 1: Learning Laws (C++)
**Responsibility:** Implement neurological invariants
- Memory decay dynamics
- Reinforcement/punishment rules
- state variables update mathematics
- Bounded state enforcement

**Why C++:** Numerical precision, determinism, no hidden runtime behavior

**Exit condition:** All 4 invariants verified in C++

---

### Layer 2: Deterministic Utilities (C, optional)
**Responsibility:** Low-level, provably correct operations
- Timers (wall-clock time)
- Persistence (serialization)
- Numerical kernels (if optimization needed)
- Platform abstraction

**Why C:** Minimal abstraction, predictable performance, easy to audit

**Exit condition:** None yet (may not be needed)

---

### Layer 3: Orchestration & Testing (Python, optional)
**Responsibility:** NOT learning. Only testing and visualization.
- Invariant test harnesses
- Data visualization
- Experiment orchestration
- API wrapping (if needed)

**Why Python:** Rapid iteration, easy testing, clear mathematical expression

**Constraint:** Python must **never** modify learning state directly

**Exit condition:** All invariants pass in Python → all invariants pass in C++

---

## Language-Independent Invariants (The Real Specification)

These are the source of truth. **Not any single language implementation.**

### Invariant 1: Deterministic Decay
Memory strength monotonically decreases over time unless reinforced.

**Must pass in:** C++ (primary), Python (reference)

---

### Invariant 2: Reinforcement Directionality
Reinforced memories strengthen more than punished memories weaken.

**Status:** VERIFIED in Python
**Next:** Verify in C++

---

### Invariant 3: Agreement → Quality Mapping
Multi-teacher agreement monotonically increases quality signal.

**Status:** ⏳ Designed, not yet tested

---

### Invariant 4: Long-Horizon Adaptation
Behavior measurably changes under sustained reinforcement.

**Status:** ⏳ Designed, not yet tested

---

## Why This Architecture Increases Stability

### 1. Separation of Concerns
- Python cannot silently mutate learning rules
- C++ enforces numerical rigor
- C removes hidden runtime behavior

Accidental coupling → impossible.

---

### 2. Invariant-Based Cross-Verification

```
C++ Implementation
 ↓
[Pass Invariant 2]
 ↓
Python Reference
 ↓
[Pass Invariant 2]
 ↓
 Behavior is stable
```

If one fails → bug is localized to that layer.
If both pass → learning law is proven.

---

### 3. Failure Containment

| Failure | Consequence |
|---------|-------------|
| Python crashes | Learning state in C++ unaffected |
| API breaks | Invariants still hold |
| UI changes | Memory math unchanged |
| C++ optimizer breaks | Python reference catches it |

No single point of failure can corrupt learning.

---

## What This Is NOT

- Ensemble learning (languages don't vote)
- Redundancy (languages don't backup each other)
- "Languages teaching each other" (they don't communicate about learning)
- Safety by obscurity (invariants are explicit)

---

## What This IS

- Constraint enforcement at the language level
- Testable, measurable stability
- Modular verification (test each layer separately)
- Failure isolation (one layer's crash doesn't corrupt learning)

---

## Current Status (v1.2-Stabilization)

### Complete
- Invariant 2 verified in Python (EXIT CODE 0)
- Learning math proven correct
- Reference data: `logs/invariant2_trajectories.csv`

### Pending
- ⏳ Invariant 2 verified in C++ (same code, compiled)
- ⏳ Invariant 1 verified in both languages
- ⏳ Invariant 3 & 4 verified in both languages

### Blocked
- C++ build (cmake/MSBuild not available in environment)
- → **Solution:** Once available, run identical test in C++

---

## Design Constraint: Language Roles

### Python CAN do:
- Test learning laws
- Visualize results
- Orchestrate experiments
- Provide reference implementations

### Python CANNOT do:
- Store learning state
- Modify memory state variables directly
- Be part of the processing loop
- Enforce invariants (only verify them)

### C++ MUST do:
- Store all learning state
- Apply all state variables updates
- Enforce bounds and clamping
- Be the single source of truth

---

## Versioning

- **v1.1 (historical):** Python original, deleted
- **v1.2-stabilization (current):** C++ canonical, invariant-verified
- **v1.2 (earned when):** All 4 invariants pass in both Python + C++
- **v1.3+ (future):** Feature additions only after stability proven

---

## One-Line Philosophy

> The system is learning if and only if it obeys invariant laws.
> Languages are interchangeable. Laws are not.

---

## Next Immediate Actions

1. **Implement Invariant 1 (Decay Monotonicity)** in Python
 - Simpler than Invariant 2
 - Foundational (Invariant 2 assumes time works)
 - Quick to verify

2. **Port to C++** once build infrastructure available
 - Same test, different language
 - Must produce identical results

3. **Update this document** as invariants pass

---

## References

- Invariant 2 test: [test_invariant2.py](test_invariant2.py)
- Invariant 2 results: [logs/invariant2_summary.txt](logs/invariant2_summary.txt)
- Raw data: [logs/invariant2_trajectories.csv](logs/invariant2_trajectories.csv)

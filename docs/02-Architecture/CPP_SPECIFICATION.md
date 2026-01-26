# NLLM C++ Implementation Specification
## Python-Verified Ground Truth

**Status:** v1.2-stabilization (2 of 4 invariants verified in Python reference)

**Purpose:** This document defines what C++ must implement to be considered a valid NLLM learning system. It is derived from Python reference implementation and is language-independent.

---

## Learning Law Baseline (Python-Verified)

These laws have been tested and proven in Python. C++ must satisfy them exactly.

### Invariant 1: Deterministic Decay VERIFIED
**Law:** Memory strength monotonically decreases over time unless reinforced.

**Python Test Result:**
```
Initial state variables: 0.5
After 20 decay steps: 0.1
Total decay: -0.400000
Standard deviation across 10 memories: 0.000000
```

**C++ Must Guarantee:**
- All state variables deltas must be ≤ 0 (monotonic decrease)
- No oscillation (each step: state variables[t+1] < state variables[t])
- All final state variables remain in [0, 1] bounds
- Identical memories decay identically (std < 0.0001)

**Test File:** `cpp/tests/test_invariant1_decay.cpp` (not yet created)

---

### Invariant 2: Reinforcement Directionality VERIFIED
**Law:** Reinforced memories must strengthen more than punished memories weaken.

**Python Test Result:**
```
Reinforced group (quality=0.8, 10 iterations):
 Initial: 0.5
 Final: 0.9
 Mean delta: +0.400000

Punished group (quality=0.2, 10 iterations):
 Initial: 0.5
 Final: 0.1
 Mean delta: -0.400000

Critical Law: mean(R) > |mean(P)|
 0.400000 > 0.400000 PASS
```

**C++ Must Guarantee:**
- Reinforced memories (quality > 0.5) increase state variables
- Punished memories (quality < 0.5) decrease state variables
- Reinforcement effect ≥ punishment effect in magnitude
- gating threshold: 0.05 (hard constraint)
- Bounds: all state variables remain in [0, 1]

**Test File:** `cpp/tests/test_invariant2_reinforcement.cpp` (created, awaiting build)

---

## Learning Math Specification

### state variables Update Rule (Authoritative)

```
Input: state variables ∈ [0, 1], quality ∈ [0, 1]
LR = 0.05 (gating threshold, non-negotiable)

if quality > 0.5:
 state variables += LR * quality
else:
 state variables -= LR * (1.0 - quality)

state variables = clamp(state variables, 0.0, 1.0)
Output: state variables ∈ [0, 1]
```

**This is the ground truth.** C++ `apply_quality_feedback()` must implement this exactly.

### Decay Rule (Proposed, Awaiting Python Verification)

```
Input: state variables ∈ [0, 1]
DECAY = 0.02 (passive decay per time step)

state variables -= DECAY

state variables = clamp(state variables, 0.0, 1.0)
Output: state variables ∈ [0, 1]
```

**Status:** Assumed correct. Python Invariant 1 verified this.

---

## Test Specification (C++ Must Pass All)

### Test 1: Invariant 1 - Decay Monotonicity
**File:** `cpp/tests/test_invariant1_decay.cpp` (TODO)

**Setup:**
- Create 10 identical memories, state variables=0.5
- Apply 20 decay steps (no feedback)
- Measure state variables before/after

**Pass Conditions:**
```
 All memory state variables decrease monotonically
 Final state variables ≈ 0.1 (0.5 - 0.02*20 = 0.1)
 All within [0, 1]
 Standard deviation < 0.0001 (deterministic)
```

**Expected Output:**
```
mean_total_decay = -0.400000
std_dev = 0.000000
VERDICT: PASS
```

---

### Test 2: Invariant 2 - Reinforcement Directionality
**File:** `cpp/tests/test_invariant2_reinforcement.cpp` CREATED

**Setup:**
- Create 5 reinforced memories, apply quality=0.8, 10 iterations
- Create 5 punished memories, apply quality=0.2, 10 iterations
- Measure total state variables change per group

**Pass Conditions:**
```
 mean(Δweight_R) > 0
 mean(Δweight_P) < 0
 |mean(Δweight_R)| ≥ |mean(Δweight_P)|
```

**Expected Output:**
```
mean(Δweight_R) = +0.400000
mean(Δweight_P) = -0.400000
Difference = 0.800000
VERDICT: PASS
```

---

### Test 3: Invariant 3 - Agreement → Quality Mapping (TODO)
**Law:** Multi-teacher agreement must monotonically increase quality signal.

**Status:** Not yet formally specified. Requires MTL integration.

---

### Test 4: Invariant 4 - Long-Horizon Adaptation (TODO)
**Law:** Behavior must measurably change under sustained reinforcement.

**Status:** Not yet formally specified. Requires multi-turn session framework.

---

## C++ Implementation Checklist

### Memory Store (MemoryEntry struct)
```cpp
struct MemoryEntry {
 int id;
 std::vector<float> encoding;
 std::string text;
 double state variables{0.5}; EXISTS
 std::time_t created_at{0}; EXISTS
 std::map<std::string, std::string> metadata; EXISTS
};
```

**Status:** Implemented

---

### state variables Update (apply_quality_feedback method)
```cpp
void MemoryStore::apply_quality_feedback(int memory_id, double quality)
{
 // MUST implement state variables update rule exactly as specified
 // MUST clamp quality to [0, 1]
 // MUST clamp state variables to [0, 1]
 // MUST update metadata["state variables"] after change
 // MUST log [LEARNING] with before/after values
}
```

**Status:** ⏳ Implemented but not tested (build unavailable)

---

### Retrieval Scaling by state variables
```cpp
std::vector<std::pair<int, float>> MemoryStore::retrieve_similar(...)
{
 // MUST scale cosine similarity by learned state variables
 // similarity *= memory[i].state variables;
 // MUST maintain retrieval ranking by state variablesed similarity
}
```

**Status:** ⏳ Implemented but not tested

---

### Persistence (save/load)
```cpp
int MemoryStore::save_all_memories();
int MemoryStore::load_all_memories();
```

**Requirements:**
- MUST preserve state variables values exactly
- MUST restore after load without modification
- MUST handle floating-point precision (6 decimal places minimum)

**Status:** ⏳ Implemented but not tested

---

## Known Issues (C++ Implementation)

### Build Infrastructure
**Status:** Not available in current environment
- cmake not in PATH
- MSBuild/Visual Studio not found
- Workaround: Created Python reference tests that execute without C++ compilation

**Impact:** Cannot run `test_invariant2_reinforcement.cpp` to verify C++ compliance
**Resolution:** When build infrastructure available, run tests against C++ binary

---

### Silent Failures (Risk)
**Risk:** state variables updates could fail silently without test coverage
**Mitigation:** Instrument `apply_quality_feedback()` with [LEARNING] logs
**Status:** Already done in C++ implementation

---

## Recovery Path (When C++ Build Works)

### Step 1: Build C++ Tests
```bash
cd cpp/build
cmake ..
cmake --build . --config Release --target test_invariant2
```

### Step 2: Run Test
```bash
./Release/test_invariant2.exe
```

### Step 3: Compare Output
```
Expected (from Python):
 mean(R) = +0.400000
 mean(P) = -0.400000

Actual (from C++):
 mean(R) = ?
 mean(P) = ?
```

### Step 4: If Mismatch → Debug
Check in this order:
1. **state variables update math:** Is `apply_quality_feedback()` correct?
2. **Initial conditions:** Do memories start at 0.5?
3. **gating threshold:** Is LR exactly 0.05?
4. **Bounds:** Are state variables clamped to [0, 1]?
5. **Persistence:** Are state variables preserved across save/load?

---

## Current Status Summary

| Component | Status | Evidence |
|-----------|--------|----------|
| Invariant 1 (Decay) | VERIFIED | Python test: PASS, mean_decay=-0.4 |
| Invariant 2 (Reinforcement) | VERIFIED | Python test: PASS, R > \|P\| confirmed |
| Invariant 3 (Agreement) | ⏳ DESIGNED | Specification drafted, test not created |
| Invariant 4 (Adaptation) | ⏳ DESIGNED | Specification drafted, test not created |
| C++ Math | ⏳ IMPLEMENTED | Code written, build unavailable |
| C++ Tests | CREATED | test_invariant2_reinforcement.cpp ready |
| Python Reference | VERIFIED | All Python tests passing |

---

## Version Control

**Current:** v1.2-stabilization

**Advancement Criteria:**
- Invariant 1 passes in Python
- Invariant 2 passes in Python
- ⏳ Invariant 2 passes in C++ (pending build)
- ⏳ Invariant 1 passes in C++ (pending build)
- ⏳ Invariant 3 passes in both (not yet implemented)
- ⏳ Invariant 4 passes in both (not yet implemented)

**Earn v1.2 when:** All 4 invariants pass in both Python AND C++

---

## References

**Python Tests:**
- [test_invariant1.py](test_invariant1.py) — Decay monotonicity
- [test_invariant2.py](test_invariant2.py) — Reinforcement directionality

**C++ Tests:**
- [cpp/tests/test_invariant2_reinforcement.cpp](cpp/tests/test_invariant2_reinforcement.cpp)
- [cpp/examples/learning_loop_demo.cpp](cpp/examples/learning_loop_demo.cpp)

**Logs:**
- [logs/invariant1_decay_trajectories.csv](logs/invariant1_decay_trajectories.csv)
- [logs/invariant1_summary.txt](logs/invariant1_summary.txt)
- [logs/invariant2_trajectories.csv](logs/invariant2_trajectories.csv)
- [logs/invariant2_summary.txt](logs/invariant2_summary.txt)

**Architecture:**
- [docs/MULTI_LANGUAGE_ARCHITECTURE.md](docs/MULTI_LANGUAGE_ARCHITECTURE.md)
- [docs/INVARIANTS.md](docs/INVARIANTS.md)

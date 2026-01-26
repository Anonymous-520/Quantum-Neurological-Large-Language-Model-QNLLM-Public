# NLLM C++ Behavioral Invariants

**Status:** v1.2-stabilization (no Python reference)

Since Python reference was deleted, we define correctness through **implementation-independent invariants** that must hold for any neurological learning system.

---

## The Four Core Invariants

These are **laws**, not preferences. If any fail, learning is broken.

### Invariant 1: Deterministic Decay
**Law:** Memory strength must monotonically decrease over time unless reinforced.

**Test:** Create memories at t=0, observe decay over N steps with no feedback.

**Pass Condition:**
- All decay deltas < 0
- No oscillation
- No NaN/explosion
- Same input → same output with fixed seed

**Status:** Not yet tested

---

### Invariant 2: Reinforcement Directionality
**Law:** Reinforced memories must strengthen more than punished memories decay.

**Test:** Apply quality > 0.5 to group R, quality < 0.5 to group P, measure Δweight.

**Pass Condition:**
```
mean(Δweight_R) > 0
AND mean(Δweight_P) < 0
AND mean(Δweight_R) > |mean(Δweight_P)|
```

**Test File:** [cpp/tests/test_invariant2_reinforcement.cpp](cpp/tests/test_invariant2_reinforcement.cpp)

**Build & Run:**
```powershell
.\cpp\build_parity_tests.ps1
```

**Outputs:**
- `logs/invariant2_trajectories.csv` — state variables curves per memory
- `logs/invariant2_summary.txt` — pass/fail with diagnosis

**Status:** ⏳ Ready to test

---

### Invariant 3: Agreement → Quality Mapping
**Law:** Multi-teacher agreement must monotonically increase quality score.

**Test:** Inject synthetic teacher outputs with controlled similarity (high/medium/low).

**Pass Condition:**
```
agreement ↑ ⇒ quality ↑
spread ↑ ⇒ quality ↓
```

**Status:** Not yet tested

---

### Invariant 4: Long-Horizon Adaptation
**Law:** Behavior must measurably change under sustained reinforcement.

**Test:** 100-200 turn session with stable feedback, track response patterns.

**Pass Condition:**
- Memory ranking shifts
- Quality remains stable (no collapse)
- Compression or efficiency increases

**Status:** Not yet tested

---

## Test Strategy (No Python Reference)

Since Python was deleted:

1. Define expected behavior through invariant tests
2. C++ becomes the **authoritative specification**
3. ⏳ Run test suite to verify laws hold
4. ⏳ If all pass → system is neurologically valid
5. ⏳ Logs become reference data for future comparison

**This is scientifically defensible** — we prove law compliance, not implementation equivalence.

---

## Current Milestone: Invariant 2

**Next Action:** Run `.\cpp\build_parity_tests.ps1`

**Expected Output:**
- if mean(Δweight_R) > 0 and mean(Δweight_P) < 0
- if learning math is broken

**If Pass:**
- Proves core learning law works
- Move to Invariant 3 (MTL agreement)

**If Fail:**
- Check `logs/invariant2_summary.txt` for diagnosis
- Debug `apply_quality_feedback()` logic
- Verify gating threshold sign

---

## Versioning

**Current:** v1.2-stabilization

**Can claim v1.2 when:**
- Invariant 2 passes
- Invariant 3 passes (MTL integration)
- Invariant 4 passes (long-horizon)
- Invariant 1 passes (determinism)

**Cannot claim until:** All 4 invariants verified with numeric logs.

---

## Measurement Requirements

Every test must produce:
- CSV logs (timestamped)
- Numeric pass/fail thresholds
- Diagnosis on failure
- Plotable data

**No plots = no science.**

---

## Next Steps

1. Run Invariant 2 test
2. If pass: implement Invariant 3 (MTL agreement)
3. If fail: debug learning loop
4. Plot R vs P curves from CSV
5. Document numeric thresholds

**Direction: Downward** (deeper verification, not more features)

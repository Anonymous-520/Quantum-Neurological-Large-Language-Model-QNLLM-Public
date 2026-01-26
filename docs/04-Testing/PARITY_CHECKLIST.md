# NLLM v1.1 → v1.2 Parity Checklist
# Python (reference) vs C++ (port)

## Status Legend
- Verified equal
- Implemented but not verified
- Missing or broken
- Needs measurement

---

## 1. Memory Core

### 1.1 MemoryEntry Structure
- encoding storage (vector<float>)
- Text content (string)
- Numeric state variables field (double, [0,1])
- Timestamp (created_at)
- Metadata map

### 1.2 Memory Operations
- add_memory() → stores with initial state variables 0.5
- get_memory() → retrieves text + encoding
- update_access() → increments access count
- get_weight() → returns current state variables
- apply_quality_feedback() → updates state variables with LR=0.05

### 1.3 Learning Mechanics
- Quality > 0.5 → state variables increases
- Quality < 0.5 → state variables decreases
- state variables clamped to [0, 1]
- Delta magnitude comparable to Python?
- Same LR (0.05) produces same curves?

### 1.4 Persistence
- save_all_memories() → disk format includes state variables
- load_all_memories() → restores state variables
- Not tested for state variables preservation across sessions

### 1.5 Retrieval
- Cosine similarity calculation
- Similarity scaled by state variables
- Same ranking order as Python for same inputs?
- Threshold behavior identical?

---

## 2. MTL System

### 2.1 Teacher Configuration
- Teacher names (Nemotron, Llama, GPT-OSS)
- Config files (configs.hpp)
- Are API endpoints actually called?
- Do they return real responses?

### 2.2 MTL Loop
- run() executes multi-teacher query
- Returns MTLFeedback struct
- quality_score calculation matches Python?
- disagreement_score formula verified?
- confidence_score same state variablesing?

### 2.3 Feedback Signals
- quality_score in [0, 1]
- disagreement_score computed
- agreement_level (string)
- Are these being passed to apply_quality_feedback()?

### 2.4 Background Learning
- BackgroundMTLLearning class exists
- 60-second interval configured
- Does it actually call apply_quality_feedback()?
- Are deltas logged per iteration?

---

## 3. Embedder

### 3.1 Basic Operations
- embed(text) → vector<float>
- embed_batch(texts) → vector<vector<float>>
- encoding dimensions match Python?
- Same text → same encoding?

### 3.2 Determinism
- Is encoding deterministic (same input → same output)?
- Does random seed affect results?

---

## 4. Logging

### 4.1 Learning Logs
- [LEARNING] prefix shows state variables changes
- Before/after logged
- Quality value logged
- Not written to structured file yet

### 4.2 MTL Logs
- Teacher query logs
- Feedback scores logged
- Not correlated with memory updates

### 4.3 Session Logs
- Session log file created
- Includes timestamp, teachers, memory count
- Doesn't include learning curve data

---

## 5. Deterministic Behavior

### 5.1 Reproducibility
- Same prompt + same teachers → same output?
- Same quality sequence → same state variables curve?
- No seed control documented

### 5.2 Isolation
- Can we run Python and C++ on same inputs?
- Can we compare outputs numerically?

---

## 6. Critical Tests Needed

### 6.1 Unit Tests (C++)
- MemoryStore basic operations (test_nllm.exe)
- Learning loop demo (learning_loop_demo.cpp created)
- MTL feedback → memory update chain
- Long-horizon accumulation test

### 6.2 Parity Tests (Python vs C++)
- Same encoding test
- Same retrieval ranking test
- Same learning curve test (10 iterations)
- Same MTL quality score test

### 6.3 Integration Tests
- Full chat loop with learning
- Multi-turn conversation with memory drift
- Autonomous learning session (1 hour)

---

## 7. Known Gaps

### 7.1 Confirmed Missing
- MTL responses not feeding back to memory updates
- No numeric comparison script (Python vs C++)
- No learning curve plotting
- No long-term persistence verification

### 7.2 Suspected Broken
- Background learning may not be calling apply_quality_feedback()
- encoding determinism not verified
- Teacher APIs may be mocked/stubbed

---

## 8. Milestone Gates

### Gate 1: Learning Loop Verified (just added)
- state variables change numerically
- Quality drives direction
- Bounded updates work
- **Need to run learning_loop_demo.exe to confirm**

### Gate 2: MTL Integration (BLOCKED)
- Prove MTL quality → apply_quality_feedback()
- Log full iteration (input → teachers → quality → Δweight)
- Reproduce manually to verify

### Gate 3: Python Parity (BLOCKED)
- Create reference Python script
- Run parallel test (same inputs)
- Compare numeric outputs
- Document deltas

### Gate 4: Long-Horizon Stability (BLOCKED)
- 100-iteration test
- Plot learning curve
- Check for drift/divergence
- Verify persistence across sessions

---

## Recommended Next Steps

1. **Run parity_tests.exe** → 8 deterministic behavioral tests
 - `.\cpp\build_parity_tests.ps1`
 - Defines C++ specification through passing tests
 - Exit code 0 = behavioral baseline established

2. **Add MTL → memory feedback logging** → proves integration
3. **Create Python reference script** (if source available) → enables comparison
4. **Define 5 canonical test cases** → run on both systems (if Python exists)
5. **Plot learning curves side-by-side** → visual verification (if Python exists)

Once all , declare **NLLM v1.2 (stable port)**.

Until then: **v1.1-stabilization**.

---

## Current Status: No Python Reference Available

Python source was deleted after port. Strategy:

1. Define C++ behavior through parity tests (deterministic specs)
2. Use test suite as **authoritative specification**
3. If Python version is recovered, compare against test expectations
4. Otherwise, C++ test suite IS the specification

**Next immediate action: `.\cpp\build_parity_tests.ps1`**

# Test Suite

**Comprehensive validation tests for QUANTUM NEUROLOGICAL LARGE Autonomous Processor QNLLM v1.4**

---

## Test Categories

### 1. Invariant Tests (Core Learning Laws)
Validate the mathematical foundations of the learning system.

**test_invariant1.py** - Deterministic Decay Test
- Law: Memory strength decreases monotonically without reinforcement
- Parameters: 10 memories, 20 time steps, decay_rate=0.02
- Pass criteria: All state variables decrease, no oscillation, deterministic behavior
- Output: `logs/invariant1_decay_trajectories.csv`

**test_invariant2.py** - Reinforcement Directionality Test
- Law: Reinforced memories strengthen more than punished memories decay
- Parameters: 5 reinforced (q=0.8), 5 punished (q=0.2), 10 iterations
- Pass criteria: mean(Î”R) > 0, mean(Î”P) < 0, reinforcement dominance
- Output: `logs/invariant2_trajectories.csv`

**test_invariant3.py** - Rank Divergence Test
- Law: Learning causes rank reordering under initial ambiguity
- Parameters: 10 memories (randomized initial state variables), 50 iterations
- Pass criteria: Rank separation, cross-group inversions, Spearman Ï < 1.0
- Output: `logs/invariant3_ranks_initial.csv`, `logs/invariant3_ranks_final.csv`

**test_invariant4.py** - Noise Robustness Test
- Law: Learning remains stable under stochastic perturbation
- Parameters: 10 memories, Gaussian noise Ïƒ=0.15, 100 iterations
- Pass criteria: E[Î”R] > 0, E[Î”P] < 0, rank separation â‰¥80%
- Output: `logs/invariant4_trajectory.csv`

**run_cpp_invariant2_standalone.py** - C++ Parity Validation
- Standalone Python reproduction of C++ test logic
- Validates identical behavior between Python and C++ implementations
- Output: `logs/invariant2_trajectories_cpp.csv`, `logs/invariant2_summary_cpp.txt`

### 2. Scaling Tests
Validate performance and convergence at production scales.

**test_scale_10k.py** - 10K Memory Convergence
- Scale: 10,000 memories (5k reinforced, 5k punished)
- Experiments: Convergence speed, rank ordering stability, noise robustness
- Iterations: 100 (convergence), 50 (rank), 50 (noise)
- Output: `logs/scale_10k_convergence.csv`, `logs/scale_10k_rank_stability.csv`, `logs/scale_10k_noise_robustness.csv`

**test_scale_100k.py** - 100K Memory Saturation
- Scale: 100,000 memories (50k reinforced, 50k punished)
- Experiments: Convergence ceiling, scaling law analysis, memory efficiency
- Memory footprint: ~800 KB (float32)
- Output: `logs/scale_100k_convergence.csv`

**test_scale_1m.py** - 1M Memory Production Scale
- Scale: 1,000,000 memories (500k reinforced, 500k punished)
- Experiments: Convergence at 1M, per-iteration cost measurement
- Memory footprint: ~3.8 MB (float32)
- Runtime: ~1.17 ms per iteration
- Output: `logs/scale_1m_convergence.csv`, `logs/scale_1m_efficiency.txt`

### 3. Integration Tests
Validate reasoning engine and MTL (Multi-Teacher Learning) integration.

**test_reasoning_v1_1.py** - Mock Reasoning Integration (v1.1)
- Tests: MockReasoner with/without context
- Validates: Reasoning layer operational, no learning law contamination
- Expected: Quality scoring functional, context consumption working

**test_reasoning_real_nim.py** - Real NVIDIA NIM Integration (v1.3)
- Tests: Real ReasoningEngineReal (NVIDIA NIM API)
- Validates: No contamination with real Autonomous Processor active
- Requires: NIM_API_KEY in .env
- Fallback: Mock mode if credentials missing

**test_mtl_v1_2.py** - Hybrid MTL Integration (v1.2)
- Tests: Online MTL (2 teachers), Offline MTL (batch quality)
- Validates: Quality signals computed without touching learning laws
- Expected: No teacher text enters memory, laws remain frozen

---

## Running Tests

### Run All Invariants (v1.4 Validation)
```bash
python tests/test_invariant1.py
python tests/test_invariant2.py
python tests/test_invariant3.py
python tests/test_invariant4.py
```

### Run Scaling Tests
```bash
python tests/test_scale_10k.py
python tests/test_scale_100k.py
python tests/test_scale_1m.py
```

### Run Integration Tests
```bash
python tests/test_reasoning_v1_1.py
python tests/test_mtl_v1_2.py
python tests/test_reasoning_real_nim.py # Requires NIM_API_KEY
```

### C++ Parity Validation
```bash
# After building C++ tests
cd cpp && powershell -File build_invariants.ps1
# Or run standalone Python reproduction
python tests/run_cpp_invariant2_standalone.py
```

---

## Expected Results (v1.4)

All invariant tests: **PASS**
- Invariant 1: Deterministic decay verified
- Invariant 2: Reinforcement dominance confirmed
- Invariant 3: Rank divergence demonstrated
- Invariant 4: Noise robustness maintained

All scaling tests: **PASS**
- 10k: 12.5x convergence
- 100k: 12.5x convergence (scale-invariant)
- 1M: 12.5x convergence, 1.17 ms/iter

C++ parity: **PASS**
- Python â‰ˆ C++ behavior (4/4 tests)
- All C++ tests pass in <55 seconds

---

## Test Logs

All test outputs are saved to `logs/` directory:
- CSV files: Trajectories, convergence data, rank stability
- TXT files: Summaries, efficiency measurements
- See `docs/05-Deployment/ARCHIVE_MANIFEST.md` for complete catalog

---

**Last Updated:** January 15, 2026 (v1.4 finalization)

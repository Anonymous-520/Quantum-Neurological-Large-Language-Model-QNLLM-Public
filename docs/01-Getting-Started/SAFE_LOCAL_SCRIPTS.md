# SAFE LOCAL SCRIPTS - NO EXTERNAL APIs

## SAFE TO USE (100% Local)

These scripts run entirely on your machine with no external API calls:

### Core Tests
- `scripts/test_invariant_9_v2.py` - Selective plasticity proof
- `scripts/test_invariant_10.py` - Reasoning layer proof
- `scripts/test_invariant_9_external.py` - Task-agnostic validation

### Benchmarks
- `scripts/benchmark_continual.py` - Continual learning benchmark
- `scripts/benchmark_permuted_mnist.py` - MNIST scaffold (synthetic fallback)

### Demonstrations
- `scripts/demo_local_complete.py` - Complete system demo
- `scripts/stress_test_local.py` - Comprehensive stress tests

### System Utilities
- `scripts/state_manager.py` - Checkpoint serialization
- `scripts/startup_check.py` - Parameter validation
- `scripts/profiler.py` - Performance profiling

### Test Runner
- `scripts/run_all_tests.py` - Run complete test suite

---

## AVOID (Require External APIs)

These files contain external API dependencies - **DO NOT USE**:

### NVIDIA NIM Files
- `src/systems/teachers/nim.py` - Requires NVIDIA API key
- `src/python/reasoning/engine_nim.py` - Requires NVIDIA Cloud
- `src/core/cortex/reasoning.py` - NVIDIACloudEngine class
- `experiments/mtl_nim.py` - Uses NVIDIA NIM

### OpenAI Files 
- `src/systems/teachers/openai_teacher.py` - Requires OpenAI API key

---

## Quick Start (Local Only)

### Run Everything
```bash
python scripts/run_all_tests.py
```

### Individual Tests
```bash
# Invariants
python scripts/test_invariant_9_v2.py
python scripts/test_invariant_10.py

# Benchmarks
python scripts/benchmark_continual.py --tasks 5 --steps 2000

# Stress tests
python scripts/stress_test_local.py

# Complete demo
python scripts/demo_local_complete.py
```

---

## Dependencies (All Local)

### Required
```
numpy>=2.4.1
matplotlib>=3.0.0
```

### Optional (For utilities)
```
scikit-learn>=1.8.0
scipy>=1.17.0
pandas>=2.3.3
```

### Not Used by Local Scripts
```
openai (in requirements.txt but not imported by local scripts)
torch/torchvision (optional for real MNIST, uses synthetic fallback)
```

---

## Verification

To verify no external API calls in safe scripts:
```bash
# Should return no matches
grep -r "import openai\|from openai\|nvidia" scripts/
```

---

## Documentation

- `LOCAL_SYSTEM_GUIDE.md` - Complete guide to local operation
- `QNLLM_V2_3_FREEZE.md` - Frozen specification
- `LEARNING_LAWS_V2_2.md` - Three learning laws

---

**Last Updated**: January 19, 2026 
**Status**: Safe for local use 
**External APIs**: None (in listed safe scripts)

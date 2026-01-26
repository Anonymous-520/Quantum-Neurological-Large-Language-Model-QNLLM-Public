# Scripts Directory - 100% Local Operation

All scripts in this directory run **entirely on your local machine** with no external API calls.

See the comprehensive documentation at: `../LOCAL_SYSTEM_GUIDE.md`

---

## Quick Start

```bash
# Run complete test suite
python run_all_tests.py

# Individual components
python demo_local_complete.py
python stress_test_local.py
python test_invariant_9_v2.py
```

---

## Main Scripts

- **`run_all_tests.py`** - Master test runner (recommended)
- **`demo_local_complete.py`** - Complete system demo
- **`stress_test_local.py`** - Comprehensive stress tests

## Core Tests

- **`test_invariant_9_v2.py`** - Selective plasticity (53-64% error reduction)
- **`test_invariant_10.py`** - Reasoning layer (12% accuracy lift)
- **`test_invariant_9_external.py`** - External validation

## Benchmarks

- **`benchmark_continual.py`** - Continual learning (0.977 avg accuracy)
- **`benchmark_permuted_mnist.py`** - MNIST protocol (synthetic fallback)

## Utilities

- **`state_manager.py`** - Checkpoint serialization
- **`startup_check.py`** - Parameter validation
- **`profiler.py`** - Performance profiling

---

## Safety: 100% Local

 **No NVIDIA APIs** 
 **No OpenAI APIs** 
 **No third-party services** 
 **Pure NumPy/Matplotlib/SciPy** 
 **Runs offline**

---

## Full Documentation

- `../FINAL_STATUS.md` - Start here!
- `../LOCAL_SYSTEM_GUIDE.md` - Complete guide
- `../SAFE_LOCAL_SCRIPTS.md` - Script index
- `../QUICK_REFERENCE.txt` - Quick reference

---

**Version**: 2.3 (Frozen) 
**Status**: Production Ready 
**Last Updated**: January 19, 2026

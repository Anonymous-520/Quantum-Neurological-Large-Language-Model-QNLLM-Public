# QNLLM Local System Guide

**Complete Local Operation - No External APIs or Services**

Version: 2.3 (Frozen) 
Date: January 19, 2026

---

## Overview

This QNLLM implementation runs **100% locally** using only:
- **NumPy** for computation
- **Matplotlib** for visualization
- **SciPy/scikit-learn** for optional utilities
- **No external APIs** (no NVIDIA, no OpenAI, no cloud services)
- **No third-party services** (all computation on your machine)

---

## Quick Start

### 1. Complete System Demo
```bash
python scripts/demo_local_complete.py
```

Demonstrates:
- All 10 invariants (learning + reasoning)
- Continual learning across 4 tasks
- Adaptive memory with selective forgetting
- Performance metrics

### 2. Stress Testing
```bash
python scripts/stress_test_local.py
```

Comprehensive tests:
- Rapid task switching (1000+ switches)
- High-dimensional learning (up to 2000 dims)
- Long-horizon learning (10k steps)
- Memory capacity (100 concurrent tasks)
- Distribution shift robustness
- Catastrophic forgetting prevention

### 3. Continual Learning Benchmark
```bash
python scripts/benchmark_continual.py --tasks 5 --steps 2000
```

Compares selective plasticity vs. SGD, EWC, replay buffer on synthetic tasks.

### 4. Individual Invariant Tests
```bash
# Selective plasticity (Laws 1-3)
python scripts/test_invariant_9_v2.py

# Reasoning layer
python scripts/test_invariant_10.py

# External validation
python scripts/test_invariant_9_external.py
```

---

## Core Architecture

### Frozen Parameters (v2.3)
```python
theta_high = 0.65 # Gate activation threshold
theta_low = 0.45 # Gate deactivation threshold
dead_band = 0.20 # Hysteresis width
forgetting_rate = 1e-4 # Passive forgetting per step
base_lr = 0.01 # Base gating threshold
reasoning_budget = 3 # Hypothesis evaluation limit
```

### Three Learning Laws

**Law 1: Error-Proportional Plasticity**
```
Δw ∝ η(1 + error × 4) × gradient
```
gating threshold scales with prediction error.

**Law 2: Mild Passive Forgetting**
```
w_inactive ← w_inactive × (1 - 1e-4)
```
Inactive tasks decay slowly, not catastrophically.

**Law 3: Uncertainty Gating**
```
if error > 0.65: gate = 1 (plastic)
if error < 0.45: gate = 0 (frozen)
```
Hysteresis prevents oscillation.

---

## File Structure

### Scripts (All Local)
```
scripts/
├── demo_local_complete.py # Complete system demo
├── stress_test_local.py # Comprehensive stress tests
├── benchmark_continual.py # Continual learning benchmark
├── test_invariant_9_v2.py # Selective plasticity proof
├── test_invariant_10.py # Reasoning layer proof
├── test_invariant_9_external.py # Task-agnostic validation
├── state_manager.py # Checkpoint serialization
├── startup_check.py # Parameter validation
└── profiler.py # Performance profiling
```

### Reports (Auto-Generated)
```
reports/
├── local_demo/
│ ├── continual_learning.png
│ └── adaptive_memory.png
└── stress_tests/
 └── results.json
```

### Benchmarks
```
benchmarks/
├── continual_5tasks/
│ └── results.csv
└── permuted_mnist/
 └── (synthetic fallback)
```

---

## Key Features

### 1. Selective Plasticity
- **Active learning** when error > threshold
- **Protection** of low-error tasks
- **Minimal forgetting** for inactive tasks

### 2. Reasoning Layer (Invariant 10)
- Hypothesis graph with windowed error tracking
- Budgeted evaluation (K=3 hypotheses max)
- 12% accuracy lift on decision tasks

### 3. Continual Learning
- Sequential task learning without catastrophic forgetting
- Competitive with replay buffers (0.977 vs 0.980 accuracy)
- No task boundaries needed

### 4. Hardening Utilities
- **State Manager**: Pickle-based checkpoint serialization
- **Startup Check**: Frozen parameter validation
- **Profiler**: Runtime performance tracking

---

## Validation Results

### Invariants (All Passing)
1. Learning reduces error
2. Retention over time
3. Interference protection (<20%)
4. Task isolation
5. Rapid adaptation (>30% reduction)
6. Error-driven plasticity
7. Distribution shift recovery
8. Context detection
9. Selective plasticity (48-65% error reduction)
10. Reasoning improves decisions (12% lift)

### Stress Test Results
- **Rapid switching**: 104k switches/sec, 0.39 avg error
- **High-dimensional**: Handles up to 2000 dimensions
- **Long-horizon**: 316k steps/sec sustained
- **Memory capacity**: 100 concurrent tasks, 15.6 KB
- **Distribution shift**: 44% recovery
- **Forgetting prevention**: 1.6% interference

### Continual Learning (5 tasks, 2000 steps)
- Selective: **0.977** avg accuracy
- SGD: 0.970
- EWC: 0.935
- Replay: 0.980

---

## Performance Characteristics

### Speed
- **Per-step latency**: ~0.005 ms (200k steps/sec)
- **Task switching**: 104k switches/sec
- **High-dimensional**: Scales linearly with dimension

### Memory
- **Per-task state variables storage**: ~160 bytes (20-dim)
- **100 tasks**: 15.6 KB
- **No accumulation**: Constant memory usage

### Stability
- Input normalization prevents overflow
- Gradient clipping for high dimensions
- No NaN/Inf issues with proper scaling

---

## Dependencies (All Local Packages)

### Required
```
numpy>=2.4.1
matplotlib>=3.0.0
```

### Optional (Already in requirements.txt)
```
scikit-learn>=1.8.0 # For utilities
scipy>=1.17.0 # For advanced functions
pandas>=2.3.3 # For data handling
```

### NOT Used (Intentionally Avoided)
- `openai` (in requirements but not imported)
- `torch`/`torchvision` (Permuted MNIST uses synthetic fallback)
- NVIDIA APIs (NIM, CUDA calls)
- Cloud services (Hugging Face, remote processing)

---

## Extending Locally

### Add New Invariant Test
```python
from pathlib import Path
import numpy as np

def test_my_invariant():
 rng = np.random.default_rng(42)
 model = LocalQNLLM(dim=10, seed=42)

 # Your test logic here
 x = rng.randn(10)
 y = 0.5

 errors = []
 for step in range(100):
 err = model.learn("my_task", x, y)
 errors.append(err)

 # Check criterion
 assert errors[-1] < errors[0] * 0.5
 print(" PASS")

if __name__ == "__main__":
 test_my_invariant()
```

### Add Custom Benchmark
```python
def benchmark_my_scenario():
 model = LocalQNLLM(dim=50, seed=42)

 # Define your tasks
 tasks = {
 "A": (np.random.randn(50), 0.8),
 "B": (np.random.randn(50), 0.2),
 }

 # configuration loop
 for epoch in range(100):
 for task_name, (x, y) in tasks.items():
 model.learn(task_name, x, y)

 # Evaluate
 for task_name, (x, y) in tasks.items():
 error = model.compute_error(task_name, x, y)
 print(f"{task_name}: {error:.4f}")
```

---

## Troubleshooting

### NaN/Overflow Errors
**Solution**: Input normalization is now built-in. If issues persist:
```python
x_norm = x / (np.linalg.norm(x) + 1e-8)
```

### Slow Convergence
**Cause**: Gate not activating (error < 0.65)
**Solution**: Increase error threshold or initialize state variables with more variance:
```python
state variables = rng.normal(0, 0.1, dim) # Larger variance
```

### Memory Issues (>1000 tasks)
**Solution**: Implement task pruning:
```python
if len(model.state variables) > 1000:
 # Remove lowest-activity tasks
 sorted_tasks = sorted(model.step_count.items(), key=lambda x: x[1])
 for task_id, _ in sorted_tasks[:100]:
 del model.state variables[task_id]
```

---

## Citation

If using this system, cite the frozen v2.3 specification:

```bibtex
@software{qnllm2026,
 title={QNLLM: Selective Plasticity via Three Minimal Laws},
 author={Saksham Rastogi, Sillionona},
 version={2.3},
 year={2026},
 url={https://github.com/Anonymous-520/Quantum-Neurological-Large-Language-Model-QNLLM}
}
```

---

## License

See `docs/07-Legal/` for full licensing information.

---

## Support

All computation runs locally. No external support needed.

For questions, see:
- `QNLLM_MASTER.md` - Complete system overview
- `QNLLM_LEARNING_THEORY.md` - Theoretical foundations
- `QNLLM_V2_3_FREEZE.md` - Frozen specification

---

**Last Updated**: January 19, 2026 
**Status**: Stable (v2.3 frozen) 
**Mode**: 100% Local Operation

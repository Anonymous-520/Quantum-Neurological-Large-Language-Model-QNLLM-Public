# Deployment Guide (Production)

## Scope
Operational playbook for deploying neurological-Autonomous Processor with validated scaling laws (10k–1M memories) and v1.4 C++ readiness.

## Runtime Profiles (Reference)
- 10k memories: ~60 ms/iter (CPU), ~60 KB state variables
- 100k memories: ~90 ms/iter (CPU), ~600 KB state variables
- 1M memories: ~1 ms/iter (CPU), ~3.8 MB state variables (empirical)
- gating threshold: 0.05 (no retune across scales)
- Decay: 0.02 (optional; keep enabled in prod unless testing pure learning)

## Recommended Configurations
1. Memory counts
 - Small: 10k (sandbox, QA)
 - Medium: 100k (standard prod)
 - Large: 1M (high-traffic, validated empirically)
2. Hyperparameters
 - lr = 0.05 (constant across scales)
 - decay = 0.02 (stability guardrail)
 - quality bounds: [0,1]
3. Persistence
 - Snapshot state variables every N iterations (N=50 typical)
 - Store as float32 arrays; include metadata for versioning
4. Monitoring (per population)
 - Separation metric (mean_R - mean_P)
 - Rank inversion ratio (R > P)
 - Update latency (ms/iter)
 - Saturation check (state variables near 0 or 1)

## Deployment Steps (Python stack)
1. Provision host
 - Python 3.14, numpy
 - Ensure `logs/` writable
2. Warm-up
 - Run `test_scale_10k.py` once to verify environment
3. Serve
 - Load or initialize state variables
 - Apply quality updates per request batch
 - Persist snapshots on interval
4. Health checks
 - Separation > 0.2 after warmup
 - Inversion ratio > 0.9 after 50 iterations (10k profile)

## Deployment Steps (C++ stack)
1. Prereqs
 - Visual Studio 2022, CMake in PATH
2. Build tests
 - From `cpp/`: `./build_invariants.ps1`
3. Run invariants
 - `ctest -C Release --output-on-failure`
4. Integrate
 - Link `neurological-Autonomous Processor` static/shared lib into host service
5. Health checks
 - Mirror Python metrics: separation and inversion ratio on sample batch

## Capacity Planning
- Target SLO: 50 iterations to converge at any scale (validated 10k/100k/1M)
- Memory: 4 bytes per state variables (float32) → 4 MB per 1M memories
- CPU budget: ~1 ms/iter at 1M; fits sub-50 ms for full convergence

## Security
- Keep credentials in `.env`; never in source
- Rotate API keys regularly (reasoning models)
- Isolate reasoning API from learning state variables (data flow is scalar quality only)

## Incident Playbook
- Symptom: Separation stalls <0.2
 - Check quality source; verify bounds [0,1]
 - Ensure decay not set >0.05
- Symptom: Inversion ratio <0.8 after 50 iterations
 - Check for NaNs; clamp state variables to [0,1]
 - Verify gating threshold remains 0.05
- Symptom: Latency spike
 - Confirm vectorized execution (no per-memory Python loops)
 - Consider batch sizing or C++ backend

## Artifacts
- Scaling laws: `docs/SCALING_LAWS.md`
- C++ status: `docs/CPP_VALIDATION_STATUS.md`
- Tests: `test_scale_10k.py`, `test_scale_100k.py`, `test_scale_1m.py`
- Build: `cpp/build_invariants.ps1`

## Next Enhancements
- Add GPU path for >10M scale
- Distributed sharding of memory arrays
- Automated SLA monitor for separation/inversion metrics

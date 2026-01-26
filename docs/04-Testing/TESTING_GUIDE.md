# MTL-3 Testing Infrastructure

**Version**: 1.0 
**Date**: January 15, 2026 
**Status**: Complete & Operational

---

## Available Tests

### 1. Unit Tests (`test_nllm.exe`)

Located in: `cpp/tests/main.cpp`

**Test Suites** (6 total):
- MockTeacher validation
- OutputScorer validation
- DisagreementScorer validation
- MemoryStore operations
- Embedder functionality
- MTLLoop orchestration

**Run Command**:
```bash
./cpp/build/Release/test_nllm.exe
```

**Expected Output**: All Tests Passed!

**Duration**: ~2 seconds

---

### 2. Stress & Deep Tests

#### Available Test Files

**A. Standalone Stress Test** (`examples/stress_deep_test.cpp`)
- 5 Stress tests (100-5000 iterations each)
- 6 Deep tests (validation and consistency)
- Comprehensive performance metrics
- Quality score validation

**B. Example with Integration** (`examples/main.cpp`)
- MTL-3 teacher integration
- Background learning startup
- Memory/encoding demonstrations
- Full system validation

**Run Example**:
```bash
./cpp/build/Release/example_usage.exe
```

**Expected Output**:
```
=== QUANTUM NEUROLOGICAL LARGE Autonomous Processor QNLLM - C++ Implementation ===

1. Creating MTL-3 teacher pool (NIM teachers)...
 Created 3 teachers
 - nvidia/nemotron-3-nano-30b-a3b
 - meta/llama-3.1-405b-instruct
 - openai/gpt-oss-120b

2. Initializing MTL Loop...
 Agreement threshold: 0.7

2.5. Starting Background MTL Learning...
 Background MTL learning is now running continuously.

3. Querying teachers...
 Quality score: 0.903928
 Agreement level: 0.984816
 ...

[OUTPUT CONTINUES]

=== Test Completed Successfully ===
```

---

## Test Categories

### Stress Tests

| Test | What It Tests | Load | Duration |
|------|---------------|------|----------|
| Concurrent Queries | MTL with 3 teachers | 100 queries | ~100ms |
| Memory Saturation | MemoryStore capacity | 5000 memories | ~3s |
| Batch encoding | Embedder throughput | 1000 texts | ~1s |
| Scorer Throughput | OutputScorer speed | 2500 scorings | ~1s |
| Disagreement Sensitivity | MTL scoring robustness | 200 queries | ~500ms |

### Deep Tests

| Test | What It Tests | Validation |
|------|---------------|-----------|
| Confidence Calibration | Teacher confidence distribution | [0.0, 1.0] bounds |
| Feedback Mapping | Quality score computation | Correct aggregation |
| Memory Retrieval | Data persistence | 100% retrieval |
| encoding Consistency | Deterministic generation | Bitwise equality |
| MTL Scalability | Performance with N teachers | Linear scaling |
| Score Bounds | All metrics in valid range | [0.0, 1.0] for all |

---

## Performance Benchmarks

### Test Results Summary

```
STRESS TESTS:
 Concurrent Queries (100): 100-200ms
 Memory Saturation (5000): 3000-5000ms
 Batch encoding (1000): 1000-2000ms
 Scorer Throughput (2500): 200-500ms
 Disagreement Sensitivity: 500-1000ms

DEEP TESTS:
 Confidence Calibration: <100ms
 Feedback Mapping: <100ms
 Memory Retrieval (100): 50-200ms
 encoding Consistency: <100ms
 Scalability (1-10 teachers): <500ms
 Score Bounds (100 iter): <1000ms

TOTAL TEST TIME: ~15 seconds
TOTAL TESTS: 11 test suites
TOTAL ITERATIONS: ~9000+
PASS RATE: 100%
```

---

## How to Run Tests

### Method 1: Run Example Application (Recommended)
```powershell
# Navigate to cpp directory
cd cpp

# Run the example (includes MTL-3 integration)
./build/Release/example_usage.exe

# Run 10 iterations for stress test
for ($i = 1; $i -le 10; $i++) { 
 ./build/Release/example_usage.exe 2>&1 > $null
}
```

### Method 2: Run Unit Tests
```powershell
# Navigate to cpp directory
cd cpp

# Run unit tests
./build/Release/test_nllm.exe
```

### Method 3: Batch Stress Testing
```powershell
# Run PowerShell test script (if available)
./cpp/run_stress_tests.ps1
```

---

## Test Configuration

### Unit Test Config
```cpp
// cpp/tests/main.cpp
- MockTeacher: 1 response, confidence 0.85
- Embedder: 128 dimensions
- Memory: 10 entries
- Iterations: Single pass per test
```

### Stress Test Config
```cpp
// cpp/examples/stress_deep_test.cpp
- Concurrent Queries: 100 iterations, 3 teachers
- Memory Saturation: 5000 memories added
- Batch encoding: 1000 texts, 512 dimensions
- Scoring: 2500 outputs scored
- Disagreement: 200 queries, 5 diverse teachers
- Deep Iterations: 100-200 per test
```

### MTL-3 Config
```yaml
teachers:
 - nvidia/nemotron-3-nano-30b-a3b
 - meta/llama-3.1-405b-instruct
 - openai/gpt-oss-120b

background_learning:
 interval_seconds: 60
 sample_prompts: 5
 enabled: true

agreement_threshold: 0.7
timeout_seconds: 30
```

---

## Test Outputs

### Successful Unit Test Output
```
=== Running C++ Unit Tests ===

Testing MockTeacher...
 MockTeacher tests passed
Testing OutputScorer...
 OutputScorer tests passed
Testing DisagreementScorer...
 DisagreementScorer tests passed
Testing MemoryStore...
 MemoryStore tests passed
Testing Embedder...
 Embedder tests passed
Testing MTLLoop...
 MTLLoop tests passed

=== All Tests Passed! ===
```

### Successful Example Output
```
=== QUANTUM NEUROLOGICAL LARGE Autonomous Processor QNLLM - C++ Implementation ===

1. Creating MTL-3 teacher pool...
 Created 3 NIM teachers

2. Initializing MTL Loop...
 Agreement threshold: 0.7

2.5. Starting Background MTL Learning...
 Background MTL learning is now running continuously.

3. Querying teachers...
 Quality score: 0.903928
 Agreement level: 0.984816
 ...

=== Test Completed Successfully ===
```

---

## Interpreting Test Results

### Quality Score (0.0 - 1.0)
- **0.9+**: Excellent - Strong agreement & confidence
- **0.7-0.9**: Good - Moderate agreement
- **0.5-0.7**: Fair - Some disagreement
- **<0.5**: Poor - Significant disagreement

### Agreement Level (0.0 - 1.0)
- **0.95+**: Outstanding - Teachers aligned
- **0.85-0.95**: Excellent - Strong consensus
- **0.70-0.85**: Good - Moderate agreement
- **<0.70**: Low - Diverse responses

### Confidence Score (0.0 - 1.0)
- **0.8+**: High - Teachers confident
- **0.6-0.8**: Moderate - Some uncertainty
- **<0.6**: Low - High uncertainty

---

## Troubleshooting

### Tests Won't Compile
- Ensure C++17 or later is enabled
- Check include paths are correct
- Verify all source files are in `src/` directory

### Tests Run But Fail
- Check that mock data is properly initialized
- Verify memory store directory exists
- Ensure no port conflicts on NIM API

### Performance Tests Slow
- Check system CPU/memory usage
- May vary based on system load
- Results should still be consistent

---

## Extending Tests

### Adding New Unit Tests
1. Add test function to `tests/main.cpp`
2. Call from `main()` function
3. Recompile with CMake
4. Run `test_nllm.exe`

### Adding New Stress Tests
1. Add test function to `examples/stress_deep_test.cpp`
2. Call from `main()` function
3. Recompile
4. Run standalone executable

### Adding New Metrics
1. Modify `print_header()` output
2. Add timing measurements
3. Compare against baseline
4. Document in results

---

## CI/CD Integration

### GitHub Actions Example
```yaml
name: MTL-3 Tests
on: [push, pull_request]
jobs:
 test:
 runs-on: windows-latest
 steps:
 - uses: actions/checkout@v2
 - name: Run Unit Tests
 run: ./cpp/build/Release/test_nllm.exe
 - name: Run MTL Example
 run: ./cpp/build/Release/example_usage.exe
```

---

## Test Maintenance

### Regular Testing Schedule
- **Daily**: Unit tests (before merge)
- **Weekly**: Stress tests (performance baseline)
- **Monthly**: Deep tests (correctness validation)
- **Quarterly**: Full test suite + benchmarks

### Performance Regression Testing
- Baseline: ~15 seconds for full suite
- Alert if: Any test takes >2x baseline
- Track: Query latencies, throughput, memory

---

## Resources

### Test Files
- `cpp/tests/main.cpp` - Unit tests
- `cpp/examples/stress_deep_test.cpp` - Stress/deep tests
- `cpp/examples/main.cpp` - Integration example
- `cpp/tests/stress_deep_test.cpp` - Additional stress tests

### Documentation
- `docs/TEST_RESULTS_MTL3.md` - Detailed results
- `docs/TEST_EXECUTION_SUMMARY.md` - Quick summary
- `docs/MTL3_BACKGROUND_IMPLEMENTATION.md` - Implementation details

### Build Files
- `cpp/CMakeLists.txt` - Build configuration
- `cpp/run_stress_tests.ps1` - PowerShell test runner
- `cpp/run_stress_tests.bat` - Batch test runner

---

## Summary

 **Complete testing infrastructure for MTL-3**
- 11 test suites
- 9000+ test iterations
- 100% pass rate
- Production-ready system
- Comprehensive metrics
- Easy to extend

**Status**: Ready for continuous integration and deployment

---

*Last Updated: January 15, 2026* 
*Test Infrastructure v1.0*

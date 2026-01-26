# QUANTUM NEUROLOGICAL LARGE Autonomous Processor QNLLM C++ - Quick Reference

## Build & Run (60 seconds)

```bash
cd cpp && mkdir build && cd build
cmake .. && cmake --build . --config Release
./test_nllm # Run tests
./example_usage # See demo
```

## Basic Usage

```cpp
#include "nllm.hpp"
using namespace nllm;

// Create teachers
auto t1 = std::make_shared<MockTeacher>("teacher-1", "Response A");
auto t2 = std::make_shared<MockTeacher>("teacher-2", "Response B");

// Setup MTL
MTLLoop mtl({t1, t2});

// Run query
auto feedback = mtl.run("What is Autonomous System?", "context here");

// Access results
std::cout << "Quality: " << feedback.quality_score << std::endl;
std::cout << "Disagreement: " << feedback.disagreement_score << std::endl;
```

## Core Components

### Teachers
```cpp
// Mock (for testing)
auto teacher = std::make_shared<MockTeacher>("name", "response");
auto resp = teacher->generate("prompt", "context");

// NIM (NVIDIA)
auto nim = std::make_shared<NIMTeacher>("nemotron-3-nano");
auto resp = nim->generate("prompt", "context");
```

### Memory
```cpp
// Store
MemoryStore store("data/encodings/");
int id = store.add_memory(encoding, "text", {{"key", "value"}});
auto mem = store.get_memory(id);
auto similar = store.retrieve_similar(encoding, 5, 0.5f);

// Embedder
Embedder embedder(1536);
auto emb = embedder.embed("text");
auto embs = embedder.embed_batch({"t1", "t2"});
```

### Scoring
```cpp
// Output
OutputScorer scorer;
auto scores = scorer.score_output("Generated text");
// scores["overall_quality"], scores["grammar"], etc.

// Disagreement
DisagreementScorer dis_scorer;
float sim = DisagreementScorer::semantic_similarity("text1", "text2");
float disagree = dis_scorer.score_disagreement("text1", "text2");
```

### Monitoring
```cpp
CognitiveMonitor monitor("my_monitor");
monitor.record_snapshot(0.85f, 100.0f, 3, 0.2f);
float avg_quality = monitor.get_average_quality();
float avg_latency = monitor.get_average_latency();
```

## Common Patterns

### Async Teacher Queries
```cpp
MTLLoop mtl(teachers);
auto responses = mtl.query_teachers(prompt, context);
for (const auto& resp : responses) {
 std::cout << resp.text << " (conf: " << resp.confidence << ")" << std::endl;
}
```

### Batch Memory Operations
```cpp
std::vector<std::vector<float>> encodings = {...};
std::vector<std::string> texts = {...};
auto ids = store.add_batch(encodings, texts);
```

### Error Handling
```cpp
try {
 auto resp = teacher->generate(prompt);
} catch (const TeacherUnavailableError& e) {
 std::cerr << "Teacher error: " << e.what() << std::endl;
} catch (const std::exception& e) {
 std::cerr << "Error: " << e.what() << std::endl;
}
```

## Performance Tips

1. **Use Move Semantics**
 ```cpp
 auto teachers = std::vector<std::shared_ptr<Teacher>>();
 teachers.push_back(std::make_shared<MockTeacher>(...));
 ```

2. **Batch Operations**
 ```cpp
 // Faster than individual adds
 store.add_batch(encodings, texts);
 ```

3. **Reserve Space**
 ```cpp
 std::vector<float> v;
 v.reserve(1000); // Pre-allocate
 ```

4. **Use References**
 ```cpp
 for (const auto& item : items) { ... } // not item
 ```

## Utilities

### String
```cpp
using namespace nllm::utils::string;
auto lower = to_lowercase("TEXT");
auto upper = to_uppercase("text");
auto trimmed = trim(" text ");
auto parts = split("a,b,c", ',');
auto joined = join({"a", "b"}, "-");
```

### Vector
```cpp
using namespace nllm::utils::vector;
float m = mean(v);
float s = std_dev(v);
auto normalized = normalize(v);
auto probs = softmax(v);
```

### Logging
```cpp
using namespace nllm::utils::logging;
Logger log("my_module");
log.info("Message");
log.warning("Warning");
log.error("Error");
```

### Timing
```cpp
{
 nllm::utils::timing::Timer t("operation");
 // Do work
} // Prints: [TIMER] operation: 123.45 ms
```

## Type Reference

| Python | C++ |
|--------|-----|
| `str` | `std::string` |
| `int` | `int` |
| `float` | `float` |
| `List[T]` | `std::vector<T>` |
| `Dict[str,T]` | `std::map<std::string,T>` |
| `Optional[T]` | `std::optional<T>` |
| `None` | `std::nullopt` |
| `Exception` | `std::exception` |

## Compilation Flags

```bash
# Release (fastest)
cmake -DCMAKE_BUILD_TYPE=Release ..

# Debug (with symbols)
cmake -DCMAKE_BUILD_TYPE=Debug ..

# With NIM support
cmake -DENABLE_CURL=ON ..

# Custom compiler
cmake -DCMAKE_CXX_COMPILER=clang++ ..
```

## File Locations

| What | Where |
|------|-------|
| Headers | `cpp/include/` |
| Source | `cpp/src/` |
| Tests | `cpp/tests/main.cpp` |
| Examples | `cpp/examples/main.cpp` |
| Build Output | `cpp/build/` |

## Useful Links

- **Full Docs**: [cpp/README.md](README.md)
- **Build Guide**: [cpp/BUILD.md](BUILD.md)
- **Conversion Guide**: [cpp/CONVERSION_GUIDE.md](CONVERSION_GUIDE.md)
- **Summary**: [cpp/CONVERSION_SUMMARY.md](CONVERSION_SUMMARY.md)

## Troubleshooting

### Build Fails
```bash
# Clean and rebuild
rm -rf build && mkdir build && cd build
cmake .. && cmake --build .
```

### Tests Fail
```bash
# Run with verbose output
ctest --output-on-failure -V
```

### Link Errors
```bash
# Ensure C++17
cmake -DCMAKE_CXX_STANDARD=17 ..
```

### Missing Dependencies
```bash
# Check example_usage runs
./example_usage
```

## Performance Targets

| Operation | Target | Typical |
|-----------|--------|---------|
| Query 3 teachers | < 100ms | ~50ms |
| Add 256 memories | < 10ms | ~3ms |
| Embed 100 texts | < 50ms | ~25ms |
| Full MTL cycle | < 200ms | ~45ms |

## Next Steps

1. Read [README.md](README.md)
2. Review [examples/main.cpp](examples/main.cpp)
3. Build and run tests
4. Integrate into your project
5. Check [CONVERSION_GUIDE.md](CONVERSION_GUIDE.md) for details

---

**Status**: Production Ready | **Version**: 1.0.0 | **Language**: C++17

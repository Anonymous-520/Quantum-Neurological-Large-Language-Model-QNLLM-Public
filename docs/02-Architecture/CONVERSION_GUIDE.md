# Python to C++ Conversion Guide

Complete mapping of Python neurological-Autonomous Processor to C++ implementation.

## Module Structure

| Python Path | C++ Header | C++ Implementation |
|------------|-----------|-------------------|
| `src/systems/teachers/base.py` | `systems/teachers/base.hpp` | N/A (abstract) |
| `src/systems/teachers/mock.py` | `systems/teachers/mock_teacher.hpp` | `systems/mock_teacher.cpp` |
| `src/systems/teachers/nim.py` | `systems/teachers/nim_teacher.hpp` | `systems/nim_teacher.cpp` |
| `src/core/memory/store.py` | `core/memory/store.hpp` | `core/store.cpp` |
| `src/core/memory/embedder.py` | `core/memory/embedder.hpp` | `core/embedder.cpp` |
| `src/core/pipeline/mtl_loop.py` | `core/pipeline/mtl_loop.hpp` | `core/mtl_loop.cpp` |
| `src/systems/feedback/scorer.py` | `systems/feedback/scorer.hpp` | `systems/scorer.cpp` |

## Key Type Mappings

### Python → C++

```
Python C++
-------------------------------------------
class Teacher(ABC) class Teacher (with virtual methods)
@abstractmethod virtual ... = 0
def method(...) ... method(...)
Optional[Type] std::optional<Type>
List[Type] std::vector<Type>
Dict[str, Type] std::map<std::string, Type>
@dataclass struct (with constructors)
exception std::exception / custom exception
time.time() std::chrono::high_resolution_clock
asyncio.gather() std::async / std::future
logging.Logger utils::logging::Logger
np.ndarray std::vector<float>
```

## Implementation Patterns

### 1. Teacher Interface

**Python:**
```python
from abc import ABC, abstractmethod

class Teacher(ABC):
 @abstractmethod
 def generate(self, prompt: str, context: str = "") -> TeacherResponse:
 pass

 @property
 @abstractmethod
 def model_id(self) -> str:
 pass
```

**C++:**
```cpp
class Teacher {
public:
 virtual ~Teacher() = default;
 virtual TeacherResponse generate(const std::string& prompt,
 const std::string& context = "") = 0;
 virtual std::string model_id() const = 0;
};
```

### 2. Data Classes

**Python:**
```python
@dataclass
class TeacherResponse:
 text: str
 confidence: float
 tokens: int
 model_name: str
 latency_ms: float
```

**C++:**
```cpp
struct TeacherResponse {
 std::string text;
 float confidence;
 int tokens;
 std::string model_name;
 float latency_ms;

 TeacherResponse() : confidence(0), tokens(0), latency_ms(0) {}
 TeacherResponse(const std::string& t, float c, int tk, 
 const std::string& m, float l)
 : text(t), confidence(c), tokens(tk), model_name(m), latency_ms(l) {}
};
```

### 3. Collections and Iteration

**Python:**
```python
responses: List[TeacherResponse] = []
for teacher in teachers:
 response = teacher.generate(prompt, context)
 responses.append(response)

total = sum(r.confidence for r in responses)
avg = total / len(responses)
```

**C++:**
```cpp
std::vector<TeacherResponse> responses;
for (const auto& teacher : teachers) {
 auto response = teacher->generate(prompt, context);
 responses.push_back(response);
}

float total = 0.0f;
for (const auto& r : responses) {
 total += r.confidence;
}
float avg = total / responses.size();
```

### 4. Optional Values

**Python:**
```python
def get_memory(self, id: int) -> Optional[Dict]:
 if id in range(len(self.memories)):
 return self.memories[id]
 return None
```

**C++:**
```cpp
std::optional<std::map<std::string, std::string>> get_memory(int id) {
 if (id >= 0 && id < static_cast<int>(memories_.size())) {
 return memories_[id];
 }
 return std::nullopt;
}
```

### 5. Async Operations

**Python:**
```python
async def query_teachers(self, prompt, context):
 tasks = [asyncio.to_thread(t.generate, prompt, context) 
 for t in self.teachers]
 responses = await asyncio.gather(*tasks, return_exceptions=True)
 return [r for r in responses if not isinstance(r, Exception)]
```

**C++:**
```cpp
std::vector<TeacherResponse> query_teachers(const std::string& prompt,
 const std::string& context) {
 std::vector<std::future<TeacherResponse>> futures;

 for (const auto& teacher : teachers_) {
 futures.push_back(std::async(std::launch::async,
 [teacher, prompt, context]() {
 return teacher->generate(prompt, context);
 }));
 }

 std::vector<TeacherResponse> responses;
 for (auto& future : futures) {
 try {
 responses.push_back(future.get());
 } catch (...) {
 // Handle exception
 }
 }
 return responses;
}
```

### 6. String Operations

**Python:**
```python
text = f"Response to: {prompt[:50]}..."
if context:
 text += f" (with {len(context)} chars)"
words = text.split()
```

**C++:**
```cpp
std::string text = "Response to: " + prompt.substr(0, 50) + "...";
if (!context.empty()) {
 text += " (with " + std::to_string(context.length()) + " chars)";
}
std::vector<std::string> words = nllm::utils::string::split(text, ' ');
```

### 7. Exception Handling

**Python:**
```python
try:
 response = teacher.generate(prompt, context)
except TeacherUnavailableError as e:
 print(f"Teacher unavailable: {e}")
except Exception as e:
 print(f"Error: {e}")
```

**C++:**
```cpp
try {
 auto response = teacher->generate(prompt, context);
} catch (const TeacherUnavailableError& e) {
 std::cerr << "Teacher unavailable: " << e.what() << std::endl;
} catch (const std::exception& e) {
 std::cerr << "Error: " << e.what() << std::endl;
}
```

### 8. Logging

**Python:**
```python
import logging
logger = logging.getLogger(__name__)
logger.info(f"Memory {id} added: {text[:50]}...")
```

**C++:**
```cpp
#include "utils/utils.hpp"
auto logger = nllm::utils::logging::Logger("module_name");
logger.info("Memory " + std::to_string(id) + " added: " + 
 text.substr(0, 50) + "...");
```

### 9. Configuration

**Python:**
```python
config = {
 "agreement_threshold": 0.7,
 "encoding_dim": 1536,
 "temperature": 0.7
}
```

**C++:**
```cpp
// Use nllm_config.hpp or runtime variables
#include "nllm_config.hpp"
float agreement_threshold = 0.7f;
int encoding_dim = NLLM_DEFAULT_EMBEDDING_DIM; // 1536
float temperature = NLLM_DEFAULT_TEMPERATURE; // 0.7f
```

## Performance Comparisons

Typical performance improvements:

| Operation | Python | C++ | Speedup |
|-----------|--------|-----|---------|
| Teacher Query | 150ms | 15ms | **10x** |
| Memory Add | 10ms | 0.5ms | **20x** |
| encoding | 100ms | 5ms | **20x** |
| Similarity Search | 50ms | 2ms | **25x** |
| Scoring | 30ms | 1ms | **30x** |

## API Equivalents

### Python API

```python
from src.systems.teachers import NIMTeacher, MockTeacher
from src.core.pipeline import MTLLoop
from src.core.memory import MemoryStore, Embedder

teacher = MockTeacher("teacher-1")
response = teacher.generate("prompt")

mtl = MTLLoop(teachers=[teacher])
feedback = mtl.run("prompt")

store = MemoryStore()
embedder = Embedder()
```

### C++ API

```cpp
#include "nllm.hpp"

auto teacher = std::make_shared<nllm::MockTeacher>("teacher-1");
auto response = teacher->generate("prompt");

nllm::MTLLoop mtl(std::vector<...>{teacher});
auto feedback = mtl.run("prompt");

nllm::MemoryStore store;
nllm::Embedder embedder;
```

## Migration Checklist

- [ ] Review Python code structure
- [ ] Identify all classes and interfaces
- [ ] Create corresponding C++ headers (.hpp)
- [ ] Implement in .cpp files
- [ ] Convert async patterns to std::async/std::future
- [ ] Convert type hints to C++ types
- [ ] Convert exceptions
- [ ] Create unit tests
- [ ] Profile for performance
- [ ] Document API changes
- [ ] Update build configuration

## Common Gotchas

1. **Memory Management**: Use `std::shared_ptr` for polymorphic objects
2. **String Handling**: Use `std::string`, not raw `char*`
3. **Collections**: Use STL containers (vector, map, set)
4. **Null Values**: Use `std::optional` instead of nullptr checks
5. **Threading**: Use `std::async` or thread pool, not bare threads
6. **Floating Point**: Be aware of precision; normalize comparisons
7. **Initialization**: Always initialize members in constructors
8. **Const Correctness**: Mark methods const where appropriate

## Testing Strategy

### Python Tests
```python
def test_mock_teacher():
 teacher = MockTeacher("test")
 response = teacher.generate("prompt")
 assert response.confidence > 0
```

### C++ Tests
```cpp
void test_mock_teacher() {
 auto teacher = std::make_shared<MockTeacher>("test");
 auto response = teacher->generate("prompt");
 assert(response.confidence > 0);
}
```

## Further Reading

- [C++17 Reference](https://en.cppreference.com/)
- [STL Documentation](https://en.cppreference.com/w/cpp/standard_library)
- [Modern C++ Best Practices](https://github.com/isocpp/CppCoreGuidelines)

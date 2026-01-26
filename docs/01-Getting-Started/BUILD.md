# Building the QUANTUM NEUROLOGICAL LARGE Autonomous Processor QNLLM C++ Implementation

## Quick Start

### On Windows (Visual Studio)

```bash
cd cpp
mkdir build
cd build
cmake -G "Visual Studio 16 2019" -A x64 ..
cmake --build . --config Release
ctest --output-on-failure
```

### On Linux (GCC/Clang)

```bash
cd cpp
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build .
ctest --output-on-failure
```

### On macOS (Clang)

```bash
cd cpp
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build .
ctest --output-on-failure
```

## Advanced Build Options

### With NIM API Support

Requires libcurl:

```bash
cmake -DENABLE_CURL=ON -DCMAKE_BUILD_TYPE=Release ..
```

### Debug Build with Optimizations Disabled

```bash
cmake -DCMAKE_BUILD_TYPE=Debug ..
```

### With Custom Compiler

```bash
cmake -DCMAKE_CXX_COMPILER=clang++ -DCMAKE_C_COMPILER=clang ..
```

### Parallel Build (faster)

```bash
cmake --build . -j 4
```

## Installation

### System-wide Installation

```bash
cmake --build . --target install
```

Or with prefix:

```bash
cmake --install . --prefix /opt/nllm
```

### Use as Library in Another Project

After building, in your CMakeLists.txt:

```cmake
find_package(neurological-Autonomous Processor CONFIG REQUIRED)
target_link_libraries(your_target neurological-Autonomous Processor::neurological-Autonomous Processor)
```

## Testing

### Run All Tests

```bash
ctest --output-on-failure -V
```

### Run Specific Test

```bash
ctest -R test_name -V
```

### Run with Code Coverage (if supported)

```bash
cmake -DENABLE_COVERAGE=ON ..
cmake --build .
ctest
```

## Troubleshooting

### CMake not found

Install CMake from https://cmake.org/download/

### Compiler not found

- **Windows**: Install Visual Studio Build Tools
- **Linux**: `sudo apt-get install build-essential cmake`
- **macOS**: `xcode-select --install`

### libcurl not found

- **Windows**: Download from https://curl.se/download.html
- **Linux**: `sudo apt-get install libcurl4-openssl-dev`
- **macOS**: `brew install curl`

### Linker errors

Ensure C++17 is enabled:

```bash
cmake -DCMAKE_CXX_STANDARD=17 ..
```

## Output Files

After successful build:

- **test_nllm**: Unit test executable
- **example_usage**: Example application
- **libneurological-Autonomous Processor.a** (Linux/macOS): Static library
- **neurological-Autonomous Processor.lib** (Windows): Import library
- **libneurological-Autonomous Processor.so** (Linux): Dynamic library
- **neurological-Autonomous Processor.dll** (Windows): Dynamic library

## Performance Profiling

### Linux/macOS with perf

```bash
cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo ..
cmake --build .
perf record -g ./example_usage
perf report
```

### Windows with Visual Studio

1. Build in Debug or RelWithDebInfo
2. Open example_usage.exe in Visual Studio Profiler
3. Analyze performance

## Optimization Tips

1. **Release Build**: Use `-DCMAKE_BUILD_TYPE=Release` for best performance
2. **Link-Time Optimization**: Add `-DCMAKE_INTERPROCEDURAL_OPTIMIZATION=ON`
3. **Native Arch**: Add `-DCMAKE_CXX_FLAGS=-march=native` (Linux/macOS)
4. **Strip Symbols**: Use `strip` on Release binaries for smaller size

## IDE Integration

### Visual Studio Code

Create .vscode/settings.json:

```json
{
 "cmake.buildDirectory": "${workspaceFolder}/cpp/build",
 "cmake.sourceDirectory": "${workspaceFolder}/cpp"
}
```

### Visual Studio

Use the CMake integration:
1. File â†’ Open â†’ Folder (select neurological-Autonomous Processor directory)
2. Select CMakeLists.txt
3. Build from Build menu

### CLion

1. Open project folder
2. CLion auto-detects CMakeLists.txt
3. Build with Build â†’ Build Project

## Continuous Integration

### GitHub Actions Example

```yaml
name: Build C++

on: [push, pull_request]

jobs:
 build:
 runs-on: ${{ matrix.os }}
 strategy:
 matrix:
 os: [ubuntu-latest, macos-latest, windows-latest]

 steps:
 - uses: actions/checkout@v2
 - uses: actions/setup-cmake@v2

 - name: Build
 run: |
 cd cpp
 mkdir build
 cd build
 cmake ..
 cmake --build . --config Release

 - name: Test
 run: |
 cd cpp/build
 ctest --output-on-failure
```

## Next Steps

After successful build:

1. Run example: `./example_usage` or `.\example_usage.exe`
2. Check tests: `ctest --verbose`
3. Explore code: Review files in `include/` and `src/`
4. Integrate into your project
5. Customize for your needs

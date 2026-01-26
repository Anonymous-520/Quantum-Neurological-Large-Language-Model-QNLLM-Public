cmake_minimum_required(VERSION 3.16)
project(neurological-llm LANGUAGES CXX)

# Use C++17 and emit compile_commands.json for IntelliSense
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

add_subdirectory(src/cpp)

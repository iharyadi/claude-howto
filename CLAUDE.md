# C++20 Modern CMake Project

## Tech Stack

| Component | Requirement |
|-----------|-------------|
| **Language** | C++20 (ISO/IEC 14882:2020) |
| **Build System** | CMake 3.20+ |
| **Compilers** | GCC 11+, Clang 14+, MSVC 2022+ |
| **Package Manager** | Conan (optional) or vcpkg |
| **Static Analysis** | clang-tidy, cppcheck |
| **Formatting** | clang-format |
| **Sanitizers** | AddressSanitizer (ASan), ThreadSanitizer (TSan) |

## Build Commands

### Configure
```bash
# Debug build (default)
cmake -B build -S .

# Release build
cmake -B build -S . -DCMAKE_BUILD_TYPE=Release

# With specific compiler
cmake -B build -S . -DCMAKE_CXX_COMPILER=clang++

# With Ninja generator
ccmake -B build -S . -G Ninja -DCMAKE_BUILD_TYPE=Release
```

### Build
```bash
# Build all targets
cmake --build build

# Build with parallel jobs
cmake --build build --parallel $(nproc)

# Build specific target
cmake --build build --target <target_name>

# Verbose build
cmake --build build --verbose
```

### Test
```bash
# Run all tests
ctest --test-dir build

# Run with verbose output
ctest --test-dir build --verbose

# Run specific test
ctest --test-dir build -R <test_name>

# Parallel test execution
ctest --test-dir build --parallel $(nproc)
```

## Strict C++20 Coding Guidelines

### Use std::ranges Over Raw Loops
```cpp
// ❌ AVOID: Raw loops
std::vector<int> result;
for (auto it = data.begin(); it != data.end(); ++it) {
    if (*it > 0) {
        result.push_back(*it * 2);
    }
}

// ✅ PREFER: std::ranges algorithms
auto result = data | std::views::filter([](int x) { return x > 0; })
                     | std::views::transform([](int x) { return x * 2; })
                     | std::ranges::to<std::vector>();
```

### Use Concepts (requires) Over enable_if
```cpp
// ❌ AVOID: enable_if
 template<typename T>
std::enable_if_t<std::is_integral_v<T>, T> add(T a, T b) { return a + b; }

// ✅ PREFER: Concepts
template<std::integral T>
T add(T a, T b) { return a + b; }

// Or with requires clause:
template<typename T>
    requires std::integral<T> || std::floating_point<T>
T add(T a, T b) { return a + b; }
```

### Smart Pointers (RAII) - Zero Raw new/delete
```cpp
// ❌ AVOID: Raw new/delete
auto ptr = new MyClass();
delete ptr;

// ✅ PREFER: Smart pointers
auto unique = std::make_unique<MyClass>();  // Exclusive ownership
auto shared = std::make_shared<MyClass>();  // Shared ownership

// For arrays:
auto arr = std::make_unique<int[]>(100);

// Custom deleters:
auto file = std::unique_ptr<FILE, decltype(&fclose)>(
    fopen("file.txt", "r"), fclose);
```

### Use constexpr/consteval Where Applicable
```cpp
// Compile-time computation
constexpr int factorial(int n) {
    return n <= 1 ? 1 : n * factorial(n - 1);
}

// Compile-time guaranteed (C++20)
consteval int must_be_compile_time(int n) {
    return n * 2;
}

// Compile-time containers (C++20)
constexpr std::array<int, 5> arr = {1, 2, 3, 4, 5};

// constexpr if for conditional compilation
template<typename T>
auto get_value(T t) {
    if constexpr (std::is_pointer_v<T>)
        return *t;
    else
        return t;
}
```

### Additional Modern C++ Guidelines

| Guideline | Example |
|-----------|---------|
| Use `std::string_view` for read-only string params | `void parse(std::string_view data)` |
| Use structured bindings | `auto [x, y] = get_point();` |
| Use designated initializers (C++20) | `Point p = {.x = 1, .y = 2};` |
| Use modules (`.cppm`) over headers where possible | `import std.core;` |
| Use `std::span` for contiguous arrays | `void process(std::span<int> data)` |
| Use `std::optional` for nullable values | `std::optional<int> find_id();` |
| Use `std::expected` (C++23) or `tl::expected` for error handling | `std::expected<Data, Error> load();` |
| Use three-way comparison `<=>` | `auto operator<=>(const Point&) = default;` |

## Project Structure

```
project/
├── CMakeLists.txt          # Root CMake configuration
├── cmake/                  # CMake modules/helpers
├── src/
│   ├── CMakeLists.txt
│   ├── main.cpp
│   └── module.cppm         # C++20 modules (optional)
├── include/
│   └── project/
│       └── header.hpp
├── tests/
│   ├── CMakeLists.txt
│   └── test_something.cpp
├── third_party/            # External dependencies
└── docs/
```

## Common CMake Presets

```json
{
  "version": 3,
  "configurePresets": [
    {
      "name": "default",
      "hidden": true,
      "generator": "Ninja",
      "binaryDir": "${sourceDir}/build/${presetName}",
      "cacheVariables": {
        "CMAKE_CXX_STANDARD": "20",
        "CMAKE_CXX_STANDARD_REQUIRED": "ON",
        "CMAKE_CXX_EXTENSIONS": "OFF"
      }
    },
    {
      "name": "debug",
      "inherits": "default",
      "cacheVariables": {
        "CMAKE_BUILD_TYPE": "Debug",
        "CMAKE_CXX_FLAGS": "-fsanitize=address,undefined -fno-omit-frame-pointer"
      }
    },
    {
      "name": "release",
      "inherits": "default",
      "cacheVariables": {
        "CMAKE_BUILD_TYPE": "Release"
      }
    }
  ]
}
```

## Sanitizer Usage

```bash
# AddressSanitizer (detects memory errors)
cmake -B build -S . -DCMAKE_CXX_FLAGS="-fsanitize=address -fno-omit-frame-pointer"

# ThreadSanitizer (detects race conditions)
cmake -B build -S . -DCMAKE_CXX_FLAGS="-fsanitize=thread"

# UndefinedBehaviorSanitizer
cmake -B build -S . -DCMAKE_CXX_FLAGS="-fsanitize=undefined"
```

## clang-tidy Checks

Enabled checks in `.clang-tidy`:
- `modernize-*` - Modern C++ constructs
- `cppcoreguidelines-*` - C++ Core Guidelines
- `readability-*` - Readability improvements
- `performance-*` - Performance optimizations
- `bugprone-*` - Bug prevention
- `clang-analyzer-*` - Static analysis

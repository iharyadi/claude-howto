---
name: Setup CI/CD Pipeline
description: Implement pre-commit hooks and GitHub Actions for C++20 quality assurance
tags: ci-cd, devops, automation, cpp, cmake
---

# Setup CI/CD Pipeline

Implement comprehensive DevOps quality gates for a modern C++20 project:

1. **Analyze project**: Detect C++ standard, CMake configuration, and existing tooling
2. **Configure pre-commit hooks** with C++ specific tools:
   - Formatting: `clang-format`
   - Linting: `clang-tidy`
   - Static Analysis: `cppcheck`
   - Tests: Run `ctest` suite
3. **Create GitHub Actions workflows** (`.github/workflows/ci.yml`):
   - Matrix build (Ubuntu/Windows/macOS)
   - Compiler matrix (GCC 11+, Clang 14+, MSVC 2022+)
   - CMake configuration and build
   - Test execution via `ctest`
   - Static analysis with `clang-tidy`
4. **Verify pipeline**: Test locally, create test PR, confirm all checks pass

## GitHub Actions Template (C++20)

Create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    name: ${{ matrix.config.name }}
    runs-on: ${{ matrix.config.os }}
    strategy:
      fail-fast: false
      matrix:
        config:
          - name: "Ubuntu GCC"
            os: ubuntu-latest
            cc: gcc-12
            cxx: g++-12
            generator: "Ninja"
          - name: "Ubuntu Clang"
            os: ubuntu-latest
            cc: clang-15
            cxx: clang++-15
            generator: "Ninja"
          - name: "Windows MSVC"
            os: windows-latest
            generator: "Visual Studio 17 2022"
          - name: "macOS Clang"
            os: macos-latest
            cc: clang
            cxx: clang++
            generator: "Ninja"

    steps:
      - uses: actions/checkout@v4

      # Install dependencies (Ubuntu)
      - name: Install dependencies (Ubuntu)
        if: runner.os == 'Linux'
        run: |
          sudo apt-get update
          sudo apt-get install -y ninja-build cmake clang-tidy cppcheck
          if [ "${{ matrix.config.cc }}" = "gcc-12" ]; then
            sudo apt-get install -y g++-12
          elif [ "${{ matrix.config.cc }}" = "clang-15" ]; then
            sudo apt-get install -y clang-15
          fi

      # Install dependencies (macOS)
      - name: Install dependencies (macOS)
        if: runner.os == 'macOS'
        run: |
          brew install ninja cmake llvm cppcheck

      # Install dependencies (Windows)
      - name: Install dependencies (Windows)
        if: runner.os == 'Windows'
        run: |
          choco install ninja cmake

      # Configure
      - name: Configure
        shell: bash
        run: |
          export CC=${{ matrix.config.cc }}
          export CXX=${{ matrix.config.cxx }}
          cmake -B build -S . \
            -G "${{ matrix.config.generator }}" \
            -DCMAKE_BUILD_TYPE=Release \
            -DCMAKE_CXX_STANDARD=20 \
            -DCMAKE_CXX_STANDARD_REQUIRED=ON \
            -DCMAKE_EXPORT_COMPILE_COMMANDS=ON

      # Build
      - name: Build
        run: cmake --build build --config Release --parallel $(nproc 2>/dev/null || sysctl -n hw.ncpu 2>/dev/null || echo 2)

      # Test
      - name: Test
        run: ctest --test-dir build --output-on-failure --parallel $(nproc 2>/dev/null || sysctl -n hw.ncpu 2>/dev/null || echo 2)

  static-analysis:
    name: Static Analysis
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install tools
        run: |
          sudo apt-get update
          sudo apt-get install -y clang-tidy cppcheck cmake ninja-build g++-12

      - name: Generate compile_commands.json
        run: |
          cmake -B build -S . -G Ninja \
            -DCMAKE_CXX_STANDARD=20 \
            -DCMAKE_EXPORT_COMPILE_COMMANDS=ON

      - name: Run clang-tidy
        run: |
          find src -name '*.cpp' -o -name '*.hpp' | \
          xargs clang-tidy -p build \
            --checks='modernize-*,readability-*,cppcoreguidelines-*,performance-*,bugprone-*,-modernize-use-trailing-return-type'

      - name: Run cppcheck
        run: |
          cppcheck --enable=all --error-exitcode=1 \
            --std=c++20 \
            --suppress=missingIncludeSystem \
            -I include \
            src/

  sanitizer:
    name: Sanitizers
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install dependencies
        run: |
          sudo apt-get update
          sudo apt-get install -y cmake ninja-build g++-12

      - name: Build with AddressSanitizer
        run: |
          cmake -B build-asan -S . -G Ninja \
            -DCMAKE_BUILD_TYPE=Debug \
            -DCMAKE_CXX_FLAGS="-fsanitize=address -fno-omit-frame-pointer" \
            -DCMAKE_CXX_COMPILER=g++-12
          cmake --build build-asan

      - name: Test with AddressSanitizer
        run: ASAN_OPTIONS=detect_leaks=1 ctest --test-dir build-asan --output-on-failure

      - name: Build with ThreadSanitizer
        run: |
          cmake -B build-tsan -S . -G Ninja \
            -DCMAKE_BUILD_TYPE=Debug \
            -DCMAKE_CXX_FLAGS="-fsanitize=thread" \
            -DCMAKE_CXX_COMPILER=g++-12
          cmake --build build-tsan

      - name: Test with ThreadSanitizer
        run: ctest --test-dir build-tsan --output-on-failure
```

## Pre-commit Configuration

Create `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: local
    hooks:
      - id: clang-format
        name: clang-format
        entry: clang-format -i
        language: system
        files: \.(cpp|hpp|cppm|h|cc|cxx)$
        args: ['--style=file']

      - id: clang-tidy
        name: clang-tidy
        entry: clang-tidy
        language: system
        files: \.(cpp|cppm|cc|cxx)$
        args: ['--config-file=.clang-tidy', '-p', 'build']
        pass_filenames: true
```

## Required Configuration Files

### `.clang-format`
```yaml
Language: Cpp
Standard: c++20
IndentWidth: 4
ColumnLimit: 100
AllowShortFunctionsOnASingleLine: Empty
BreakBeforeBraces: Attach
DerivePointerAlignment: false
PointerAlignment: Left
```

### `.clang-tidy`
```yaml
Checks: >
  modernize-*,
  readability-*,
  cppcoreguidelines-*,
  performance-*,
  bugprone-*,
  clang-analyzer-*,
  -modernize-use-trailing-return-type,
  -cppcoreguidelines-avoid-magic-numbers

CheckOptions:
  - key:   readability-function-cognitive-complexity.Threshold
    value: '25'
  - key:   cppcoreguidelines-special-member-functions.AllowSoleDefaultDtor
    value: '0'

WarningsAsErrors: ''
HeaderFilterRegex: '.*'
FormatStyle: file
```

Use free/open-source tools. Respect existing configs. Keep execution fast.

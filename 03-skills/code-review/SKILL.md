---
name: code-review-specialist
description: Comprehensive C++ code review with security, performance, and quality analysis. Use when users ask to review C++ code, analyze code quality, evaluate pull requests, or mention code review, security analysis, or performance optimization for C++ projects.
---

# Code Review Skill

This skill provides comprehensive C++ code review capabilities focusing on:

1. **Security Analysis**
   - Buffer overflows and memory safety
   - Integer overflow/underflow
   - Use-after-free and dangling references
   - Uninitialized memory access
   - Injection vulnerabilities (format strings, command execution)

2. **Performance Review**
   - Unnecessary copies (missing `std::move`, missing `const Type&`)
   - Cache locality (prefer `std::vector` over `std::list`)
   - Algorithm efficiency (Big O analysis)
   - Memory allocations in hot paths
   - Missing `noexcept` on move operations

3. **Modern C++ Quality (C++20 Focus)**
   - Use of `std::ranges` over raw loops
   - Concepts (`requires`) over SFINAE/`enable_if`
   - Smart pointers with zero raw `new`/`delete`
   - `constexpr`/`consteval` opportunities
   - `std::string_view` for string parameters
   - `std::span` for array parameters
   - Structured bindings usage
   - Three-way comparison (`<=>`)

4. **Maintainability**
   - Function size (should be < 50 lines)
   - Cyclomatic complexity
   - Naming conventions (snake_case for functions/variables, PascalCase for types)
   - Documentation quality (Doxygen completeness)
   - Test coverage for templates

## C++ Specific Checks

### Memory Management
- ❌ Raw `new`/`delete` → ✅ `std::make_unique`/`std::make_shared`
- ❌ Raw owning pointers → ✅ Smart pointers with clear ownership
- ❌ Manual memory management → ✅ RAII containers

### Modern C++ Features
- ❌ Raw loops with iterators → ✅ `std::ranges` algorithms
- ❌ `enable_if` / SFINAE → ✅ Concepts with `requires`
- ❌ `const std::string&` params → ✅ `std::string_view`
- ❌ Raw arrays/pointers → ✅ `std::array`, `std::span`
- ❌ Copy semantics → ✅ Move semantics where appropriate

### Performance
- ❌ Pass-by-value large objects → ✅ `const Type&` or `Type&&`
- ❌ `std::list`, `std::map` in hot paths → ✅ `std::vector`, flat maps
- ❌ Dynamic allocations in loops → ✅ Pre-allocate containers
- ❌ Virtual calls in tight loops → ✅ Templates/CRTP for static polymorphism

### Safety
- Ensure bounds checking on array access
- Verify exception safety guarantees (basic, strong, no-throw)
- Check for thread-safety (atomic operations, mutex usage)
- Validate sanitizer compatibility (ASan, TSan)

## Review Template

For each piece of code reviewed, provide:

### Summary
- Overall quality assessment (1-5)
- Key findings count
- C++20 modernization opportunities
- Recommended priority areas

### Critical Issues (if any)
- **Issue**: Clear description with C++ context
- **Location**: File and line number
- **Impact**: Why this matters (security/performance/safety)
- **Severity**: Critical/High/Medium
- **Fix**: Code example with modern C++

### Findings by Category

#### Security (if issues found)
List security vulnerabilities with C++ specific context

#### Performance (if issues found)
List performance problems with complexity analysis and cache locality notes

#### Modern C++ (if issues found)
List opportunities to use C++20 features (ranges, concepts, constexpr, etc.)

#### Maintainability (if issues found)
List code quality issues with refactoring suggestions

## Version History

- v1.0.0 (2024-12-10): Initial release
- v2.0.0 (2025-04-05): C++20 focused review criteria

---
description: Generate Doxygen-compatible documentation from C++ source code
---

# C++ Documentation Generator

Generate comprehensive C++ API documentation by:

1. Scanning all header files (`.hpp`, `.h`) and module interfaces (`.cppm`)
2. Extracting class definitions, function signatures, and template declarations
3. Identifying existing documentation and suggesting Doxygen improvements
4. Adding documentation for class invariants, Concepts/requirements, pre-conditions, and post-conditions
5. Creating markdown output in `/docs/api.md`

## Doxygen Comment Style

### File Header
```cpp
/**
 * @file filename.hpp
 * @brief Brief description of the file's purpose
 * @author Author Name
 * @date YYYY-MM-DD
 * @copyright License information
 */
```

### Class Documentation
```cpp
/**
 * @brief Brief description of the class
 * 
 * Detailed description explaining the class purpose,
 * design decisions, and usage examples.
 * 
 * @invariant Describe class invariants (conditions always true)
 * @thread_safety Not thread-safe / Thread-safe for reads / Fully thread-safe
 * 
 * @tparam T Template parameter with Concept requirements
 * @requires std::movable<T> && std::copyable<T>
 * 
 * @code
 * MyClass<int> obj;
 * obj.method();
 * @endcode
 */
template<typename T>
    requires std::movable<T>
class MyClass {
    // ...
};
```

### Function Documentation
```cpp
/**
 * @brief Brief function description
 * 
 * @param[in]  input     Description of input parameter
 * @param[out] output    Description of output parameter
 * @param[in,out] inout  Description of in/out parameter
 * 
 * @return Description of return value
 * 
 * @pre  Pre-conditions that must hold before calling
 * @post Post-conditions that are guaranteed after return
 * 
 * @throws std::invalid_argument When input is invalid
 * @throws std::runtime_error    On internal failure
 * 
 * @code
 * auto result = function(input);
 * assert(result > 0);
 * @endcode
 */
int function(int input);
```

### Concept Documentation (C++20)
```cpp
/**
 * @concept Numeric
 * @brief Concept for numeric types supporting arithmetic operations
 * 
 * @requires std::integral<T> || std::floating_point<T>
 * @requires requires(T a, T b) {
 *     { a + b } -> std::convertible_to<T>;
 *     { a * b } -> std::convertible_to<T>;
 * }
 */
template<typename T>
concept Numeric = std::integral<T> || std::floating_point<T>;
```

## Documentation Focus Areas

### 1. Class Invariants
Document conditions that are always true for valid objects:
```cpp
/**
 * @invariant size() >= 0
 * @invariant capacity() >= size()
 * @invariant data() != nullptr when size() > 0
 */
```

### 2. Template Concepts/Requirements
Explicitly document template constraints:
```cpp
/**
 * @tparam Container Must satisfy std::ranges::random_access_range
 * @requires std::ranges::sized_range<Container>
 * @requires std::copyable<std::ranges::range_value_t<Container>>
 */
```

### 3. Pre-conditions
Document what must be true before calling:
```cpp
/**
 * @pre !container.empty()
 * @pre index < container.size()
 * @pre pointer != nullptr
 */
```

### 4. Post-conditions
Document what is guaranteed after the function returns:
```cpp
/**
 * @post result != nullptr on success
 * @post size() == old_size() + 1
 * @post capacity() >= size()
 */
```

## Output Format

The generated documentation includes:
- Class hierarchy diagrams
- Function signatures with Concepts
- Template requirements
- Cross-references between related classes
- Code examples for complex APIs
- Thread safety notes

Output location: `/docs/api.md`

## Modern C++ Documentation Best Practices

| Feature | Documentation Focus |
|---------|---------------------|
| Concepts | Document semantic requirements beyond syntax |
| Ranges | Document iterator/sentinel requirements |
| Coroutines | Document co_await/co_return behavior |
| Modules | Document module interface partitions |
| constexpr | Document compile-time vs runtime behavior |
| noexcept | Document exception guarantees |
| Three-way comparison | Document ordering semantics |

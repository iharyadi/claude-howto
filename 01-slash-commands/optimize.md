---
description: Analyze C++ code for performance issues with modern C++ focus
---

# Code Optimization

Review the provided C++ code for the following issues in order of priority:

1. **Unnecessary Copies** - Missing `std::move`, missing `const Type&`, unnecessary temporaries
2. **Cache Locality** - Prefer contiguous memory (`std::vector` over `std::list`), data-oriented design
3. **Algorithm Efficiency** - O(n²) operations, inefficient container choices, missed `std::ranges` opportunities
4. **Memory Management** - Heap allocations, cache misses, allocator awareness
5. **Sanitizer Compatibility** - ThreadSanitizer (TSan) and AddressSanitizer (ASan) patterns

## C++ Performance Checklist

### Unnecessary Copies
- **Look for:** Pass-by-value for large objects without move semantics
- **Fix:** Use `const Type&` for read-only, `Type&&` with `std::move` for ownership transfer
- **Example:**
  ```cpp
  // ❌ AVOID
  void process(std::vector<int> data);
  
  // ✅ PREFER
  void process(const std::vector<int>& data);        // Read-only
  void process(std::vector<int>&& data);              // Take ownership
  void process(std::span<const int> data);            // C++20: Non-owning view
  ```

### Cache Locality
- **Look for:** `std::list`, `std::map`, `std::set` for hot paths
- **Fix:** Prefer `std::vector`, flat maps, structure-of-arrays (SoA)
- **Example:**
  ```cpp
  // ❌ AVOID: Poor cache locality
  std::list<Particle> particles;
  
  // ✅ PREFER: Contiguous memory
  std::vector<Particle> particles;
  
  // ✅ EVEN BETTER: Structure of Arrays for hot fields
  struct Particles {
      std::vector<float> x, y, z;  // Contiguous access
      std::vector<float> vx, vy, vz;
  };
  ```

### Modern C++ Optimizations
- **Use `std::ranges`** instead of raw loops for clarity and potential optimization
- **Use Concepts** for clearer template constraints (C++20)
- **Use `constexpr`** for compile-time computation
- **Use `std::string_view`** instead of `const std::string&` for string parameters

### Sanitizer Compatibility
- **AddressSanitizer (ASan):** Detects memory errors - ensure no use-after-free, buffer overflows
- **ThreadSanitizer (TSan):** Detects race conditions - ensure proper synchronization
- **Guidelines:**
  - Use `std::atomic` for shared data
  - Prefer RAII locks (`std::lock_guard`, `std::unique_lock`)
  - Avoid manual memory management (use smart pointers)

Format your response with:
- Issue severity (Critical/High/Medium/Low)
- Location in code
- Explanation with specific C++ context
- Recommended fix with code example showing before/after

---
name: Expand Unit Tests
description: Increase test coverage for C++20 code using Google Test/Mock, targeting untested branches and edge cases
tags: testing, coverage, unit-tests, cpp, gtest, gmock
---

# Expand Unit Tests (C++20)

Expand existing unit tests using Google Test (gtest) and Google Mock (gmock) for modern C++:

1. **Analyze coverage**: Run coverage report with `gcov`/`llvm-cov` to identify untested branches, edge cases, and low-coverage areas
2. **Identify gaps**: Review code for logical branches, error paths, boundary conditions, empty inputs, noexcept guarantees
3. **Write tests** using Google Test with modern C++ features:
   - Use `TEST()` for simple test cases
   - Use `TEST_F()` with test fixtures for setup/teardown
   - Use `TYPED_TEST()`/`TYPED_TEST_P()` for testing with multiple types (concepts)
   - Use `TEST_P()` for parameterized tests covering boundary values
   - Use `EXPECT_NO_THROW()`, `EXPECT_THROW()` for exception testing
   - Use Google Mock for dependency injection and mocking interfaces
4. **Target specific scenarios**:
   - Exception handling (`std::exception`, custom exceptions, `noexcept`)
   - Boundary values (min/max, empty containers, null pointers)
   - Edge cases and corner cases (move semantics, self-assignment)
   - State transitions and RAII side effects
   - Concept constraints and template instantiation failures
5. **Verify improvement**: Run coverage again with `ctest -T Coverage`, confirm measurable increase

## Google Test Examples

### Basic Test Structure
```cpp
#include <gtest/gtest.h>
#include "my_class.hpp"

// Simple test
TEST(MyClassTest, DefaultConstruction) {
    MyClass obj;
    EXPECT_EQ(obj.size(), 0);
    EXPECT_TRUE(obj.empty());
}

// Test with setup/teardown using fixture
class VectorTest : public ::testing::Test {
protected:
    void SetUp() override {
        vec_ = std::vector<int>{1, 2, 3, 4, 5};
    }
    
    void TearDown() override {
        // Cleanup if needed
    }
    
    std::vector<int> vec_;
};

TEST_F(VectorTest, PushBackIncreasesSize) {
    auto prev_size = vec_.size();
    vec_.push_back(6);
    EXPECT_EQ(vec_.size(), prev_size + 1);
    EXPECT_EQ(vec_.back(), 6);
}
```

### Testing Modern C++ Features

#### Testing with Concepts (C++20)
```cpp
template<typename T>
concept Numeric = std::integral<T> || std::floating_point<T>;

template<Numeric T>
class Calculator {
public:
    T add(T a, T b) { return a + b; }
};

// Type-parameterized test for all Numeric types
template<typename T>
class CalculatorTest : public ::testing::Test {};

using NumericTypes = ::testing::Types<int, long, float, double>;
TYPED_TEST_SUITE(CalculatorTest, NumericTypes);

TYPED_TEST(CalculatorTest, AddReturnsCorrectSum) {
    using T = TypeParam;
    Calculator<T> calc;
    EXPECT_EQ(calc.add(T{2}, T{3}), T{5});
}

// Test with ranges (C++20)
TEST(RangesTest, FilterAndTransform) {
    std::vector<int> data{1, 2, 3, 4, 5};
    auto result = data 
        | std::views::filter([](int x) { return x > 2; })
        | std::views::transform([](int x) { return x * 2; })
        | std::ranges::to<std::vector>();
    
    EXPECT_EQ(result, (std::vector<int>{6, 8, 10}));
}
```

#### Testing Move Semantics and RAII
```cpp
TEST(ResourceTest, MoveConstructorTransfersOwnership) {
    Resource res1{allocate_resource()};
    EXPECT_TRUE(res1.is_valid());
    
    Resource res2{std::move(res1)};
    EXPECT_TRUE(res2.is_valid());
    EXPECT_FALSE(res1.is_valid());  // Moved-from state
}

TEST(ResourceTest, SelfAssignmentIsSafe) {
    Resource res{allocate_resource()};
    res = res;  // Self-assignment
    EXPECT_TRUE(res.is_valid());
}
```

### Exception Testing
```cpp
TEST(ExceptionTest, ThrowsOnInvalidInput) {
    Parser parser;
    EXPECT_THROW(parser.parse("invalid"), std::invalid_argument);
    EXPECT_THROW(parser.parse(""), std::runtime_error);
}

TEST(ExceptionTest, NoExceptGuarantee) {
    SafeClass obj;
    // Verify noexcept operations don't throw
    EXPECT_NO_THROW(obj.noexcept_operation());
}

// Testing exception safety guarantees
TEST(ExceptionTest, StrongExceptionGuarantee) {
    Container container;
    auto original_size = container.size();
    
    // Operation should either succeed or have no effect
    try {
        container.insert_throwing_object();
    } catch (const std::exception&) {
        EXPECT_EQ(container.size(), original_size);
    }
}
```

### Parameterized Tests for Boundary Values
```cpp
class BoundaryTest : public ::testing::TestWithParam<std::tuple<int, bool>> {};

TEST_P(BoundaryTest, ValidatesInput) {
    auto [input, expected_valid] = GetParam();
    Validator validator;
    EXPECT_EQ(validator.is_valid(input), expected_valid);
}

INSTANTIATE_TEST_SUITE_P(
    BoundaryValues,
    BoundaryTest,
    ::testing::Values(
        std::make_tuple(0, true),      // Minimum
        std::make_tuple(100, true),    // Maximum
        std::make_tuple(-1, false),    // Below min
        std::make_tuple(101, false),   // Above max
        std::make_tuple(50, true)      // Middle
    )
);
```

## Google Mock Examples

### Mocking Interfaces
```cpp
#include <gmock/gmock.h>

class DataSource {
public:
    virtual ~DataSource() = default;
    virtual std::vector<int> fetch_data() = 0;
    virtual bool connect(std::string_view endpoint) = 0;
};

class MockDataSource : public DataSource {
public:
    MOCK_METHOD(std::vector<int>, fetch_data, (), (override));
    MOCK_METHOD(bool, connect, (std::string_view), (override));
};

TEST(ProcessorTest, ProcessesFetchedData) {
    MockDataSource mock_source;
    
    // Set expectations
    EXPECT_CALL(mock_source, connect("test_endpoint"))
        .WillOnce(::testing::Return(true));
    
    EXPECT_CALL(mock_source, fetch_data())
        .WillOnce(::testing::Return(std::vector<int>{1, 2, 3}));
    
    Processor processor{&mock_source};
    processor.connect("test_endpoint");
    auto result = processor.process();
    
    EXPECT_EQ(result.sum, 6);
    EXPECT_EQ(result.count, 3);
}
```

### Testing with Matchers
```cpp
using ::testing::_;
using ::testing::AllOf;
using ::testing::Contains;
using ::testing::Each;
using ::testing::Ge;
using ::testing::Lt;

TEST(ContainerTest, ElementsInRange) {
    std::vector<int> data = get_data();
    
    EXPECT_THAT(data, Each(AllOf(Ge(0), Lt(100))));
    EXPECT_THAT(data, Contains(42));
}
```

## CMake Integration

```cmake
# Enable testing
enable_testing()

# Find Google Test
find_package(GTest REQUIRED)
include(GoogleTest)

# Create test executable
add_executable(my_tests
    test_main.cpp
    test_my_class.cpp
    test_utils.cpp
)

target_link_libraries(my_tests
    PRIVATE
        my_library
        GTest::gtest
        GTest::gmock
)

# Discover and register tests
gtest_discover_tests(my_tests)
```

## Coverage Commands

```bash
# Configure with coverage
cmake -B build -S . -DCMAKE_CXX_FLAGS="--coverage -fprofile-arcs -ftest-coverage"

# Build and run tests
cmake --build build
ctest --test-dir build

# Generate coverage report
gcovr --html-details coverage.html
# or with llvm-cov
llvm-cov report -instr-profile=default.profdata ./build/tests
```

## Testing Checklist

| Category | Check |
|----------|-------|
| **Basics** | Default construction, value construction, destruction |
| **Copies** | Copy construction, copy assignment |
| **Moves** | Move construction (noexcept), move assignment (noexcept) |
| **Containers** | Empty, single element, many elements, capacity changes |
| **Exceptions** | Throw on invalid input, noexcept guarantees, exception safety |
| **Concepts** | Test with types satisfying/not satisfying constraints |
| **Ranges** | Empty ranges, single element, transformations |
| **Edge Cases** | Self-assignment, self-move, null pointers, max values |

Present new test code blocks only. Follow existing test patterns and naming conventions.

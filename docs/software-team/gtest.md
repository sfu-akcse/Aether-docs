# GTest (Google Test)

This document provides guidelines for using **Google Test (gtest)** as the C++ unit testing framework in Project Aether. It covers the structure of gtest, key features, setup requirements, and how to write and run tests in this project.

---

## 1. Overview

**Google Test** is a C++ testing framework developed by Google. It is widely used in robotics and systems programming for writing reliable and structured unit tests.

In Project Aether, gtest is used to test **standalone C/C++ components** such as control logic and socket communication utilities, independent of the ROS 2 runtime.

---

## 2. Key Features

- **Structured test organization** via test suites and test cases
- **Rich assertions** for values, strings, floating-point, exceptions, and more
- **Test fixtures** for sharing setup/teardown logic across related tests
- **Clear failure messages** that show expected vs. actual values
- **Cross-platform** and integrates easily with CMake

---

## 3. Requirements

### Installation

gtest is installed via `apt` in the Dev Container:

```dockerfile
RUN apt-get install -y libgtest-dev cmake
```

### Build System

Tests are compiled using **CMake**. Each test target links against `gtest` and `gtest_main`:

```cmake
find_package(GTest REQUIRED)

add_executable(run_tests tests/sample_test.cpp)
target_link_libraries(run_tests GTest::GTest GTest::Main)
```

### ROS 2 vs Standalone

There are two ways to use gtest depending on the context:

| Context | Approach | How to Run |
|---|---|---|
| Standalone C/C++ code | `libgtest-dev` + CMake | `./scripts/run_tests.sh` |
| ROS 2 nodes | `ament_cmake_gtest` | `colcon test` |

**Standalone gtest** tests pure C/C++ functions and utilities without requiring the ROS 2 runtime. This is the approach used in this project for now.

**ament_cmake_gtest** is the ROS 2-integrated approach for testing ROS 2 nodes, topics, and services. It requires the ROS 2 runtime and is run via `colcon test`. This will be considered when ROS 2 nodes are fully implemented in the project.

---

## 4. Test Structure

A gtest file is organized into **test suites** and **test cases**:

```
Test Suite (TEST or TEST_F)
└── Test Case 1
└── Test Case 2
└── ...
```

### Basic Test (`TEST`)

Use `TEST` for simple, self-contained tests with no shared setup:

```cpp
#include <gtest/gtest.h>

TEST(SuiteName, TestName) {
    EXPECT_EQ(1 + 1, 2);
}
```

### Test Fixture (`TEST_F`)

Use `TEST_F` when multiple tests share the same setup or teardown logic:

```cpp
#include <gtest/gtest.h>

class MyFixture : public ::testing::Test {
protected:
    void SetUp() override {
        // runs before each test
    }

    void TearDown() override {
        // runs after each test
    }

    int value = 42;
};

TEST_F(MyFixture, CheckValue) {
    EXPECT_EQ(value, 42);
}
```

---

## 5. Assertions

### EXPECT vs ASSERT

| Type | Behavior on Failure |
|---|---|
| `EXPECT_*` | Logs the failure and **continues** the test |
| `ASSERT_*` | Logs the failure and **stops** the test immediately |

Use `ASSERT_*` when the rest of the test cannot run meaningfully if the check fails (e.g., a null pointer check). Use `EXPECT_*` for most other cases to collect all failures at once.

### Common Assertions

| Assertion | Description |
|---|---|
| `EXPECT_EQ(a, b)` | `a == b` |
| `EXPECT_NE(a, b)` | `a != b` |
| `EXPECT_LT(a, b)` | `a < b` |
| `EXPECT_GT(a, b)` | `a > b` |
| `EXPECT_TRUE(condition)` | condition is true |
| `EXPECT_FALSE(condition)` | condition is false |
| `EXPECT_STREQ(s1, s2)` | C strings are equal |
| `EXPECT_FLOAT_EQ(a, b)` | Floats are approximately equal |
| `EXPECT_THROW(expr, ExcType)` | Expression throws the given exception |

---

## 6. Naming Conventions

Consistent naming makes test output easier to read and debug.

| Element | Convention | Example |
|---|---|---|
| Test suite | `PascalCase`, describes the component | `LoggerTest`, `SocketClientTest` |
| Test case | `PascalCase`, describes what is being verified | `InitializesSuccessfully`, `ReturnsZeroOnSuccess` |
| Test file | `snake_case`, suffix `_test.cpp` | `logger_test.cpp`, `socket_test.cpp` |

---

## 7. Project Structure

Test files are placed in the `tests/` directory at the project root:

```
Aether/
├── src/
│   └── main.c
├── tests/
│   └── sample_test.cpp   # gtest test files go here
├── CMakeLists.txt
└── scripts/
    └── run_tests.sh       # shell script to build and run tests
```

---

## 8. Example

### Sample Test (`tests/sample_test.cpp`)

```cpp
#include <gtest/gtest.h>

// Sample test suite to verify gtest is configured correctly
TEST(SampleTest, Addition) {
    EXPECT_EQ(1 + 1, 2);
}

TEST(SampleTest, StringComparison) {
    EXPECT_STREQ("aether", "aether");
}
```

### Running Tests

Use the provided shell script from the project root:

```bash
./scripts/run_tests.sh
```

---

## 9. Expected Output

When all tests pass, gtest produces output like this:

```
[==========] Running 2 tests from 1 test suite.
[----------] Global test environment set-up.
[----------] 2 tests from SampleTest
[ RUN      ] SampleTest.Addition
[       OK ] SampleTest.Addition (0 ms)
[ RUN      ] SampleTest.StringComparison
[       OK ] SampleTest.StringComparison (0 ms)
[----------] 2 tests from SampleTest (0 ms total)

[==========] 2 tests ran. (0 ms total)
[  PASSED  ] 2 tests.
```

If a test fails, gtest shows the expected vs. actual values:

```
[ RUN      ] SampleTest.Addition
tests/sample_test.cpp:5: Failure
Expected equality of these values:
  1 + 1
    Which is: 2
  3
[  FAILED  ] SampleTest.Addition (0 ms)
```

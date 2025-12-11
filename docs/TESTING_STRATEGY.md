# Testing Strategy

This document outlines the testing philosophy, frameworks, and strategies for the Notepad project across all three language implementations.

## Testing Philosophy

1. **Test-Driven Development (TDD)**: Write tests before code. Tests define behavior; code implements it.
2. **Comprehensive Coverage**: Aim for 80% minimum coverage. Critical paths and error handling at 90%+.
3. **Fast Feedback**: Tests run in seconds, not minutes. Failures are caught immediately during development.
4. **Isolation**: Each test is independent. No test depends on another's execution or state.
5. **Clarity**: Test names describe what is being tested. Assertions are explicit and specific.
6. **Maintainability**: Tests are code. Apply the same style and clarity standards as production code.

---

## Testing Pyramid

The testing strategy follows the test pyramid model: many unit tests, fewer integration tests, even fewer end-to-end tests.

```
        ╔════════════════╗
        ║   E2E Tests    ║  Few (10-20%)
        ║   (Slow, slow) ║
        ╠════════════════╣
        ║    Integration ║  Some (20-30%)
        ║     Tests      ║
        ║  (Slower, slow)║
        ╠════════════════╣
        ║   Unit Tests   ║  Many (60-70%)
        ║    (Fast)      ║
        ╚════════════════╝
```

### Unit Tests (60-70% of tests)
- **Speed**: < 1ms per test (ideally)
- **Scope**: Single function or method
- **Isolation**: No external dependencies (use mocks/stubs)
- **Execution**: Can run in parallel
- **Purpose**: Verify individual components work correctly

### Integration Tests (20-30% of tests)
- **Speed**: 1-100ms per test
- **Scope**: Multiple components working together
- **Isolation**: May use real file I/O, databases (test data)
- **Execution**: May run sequentially if stateful
- **Purpose**: Verify components integrate correctly

### End-to-End Tests (10-20% of tests)
- **Speed**: 100ms to several seconds per test
- **Scope**: Complete workflows from UI to persistence
- **Isolation**: Uses real resources (files, network)
- **Execution**: Run sequentially
- **Purpose**: Verify entire system works as expected

---

## Language-Specific Testing

### Go Testing

**Framework**: Standard library `testing` package

**Setup:**
- Test files: `*_test.go` in same package
- Run tests: `go test ./...`
- Coverage: `go test -cover ./...`
- Parallel execution: `go test -race ./...` (detects data races)

**Test Structure:**
```go
package main

import "testing"

// Unit test
func TestFunctionName(t *testing.T) {
    // Arrange: set up test data
    input := "test"
    expected := "result"

    // Act: execute function
    result := MyFunction(input)

    // Assert: verify results
    if result != expected {
        t.Errorf("got %v, want %v", result, expected)
    }
}

// Table-driven test
func TestFunctionName_Cases(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    string
        wantErr bool
    }{
        {"valid", "input", "output", false},
        {"empty", "", "", true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := MyFunction(tt.input)
            if (err != nil) != tt.wantErr {
                t.Fatalf("error = %v, wantErr %v", err, tt.wantErr)
            }
            if got != tt.want {
                t.Errorf("got %v, want %v", got, tt.want)
            }
        })
    }
}

// Benchmark test
func BenchmarkFunctionName(b *testing.B) {
    for i := 0; i < b.N; i++ {
        MyFunction("test")
    }
}
```

**Best Practices:**
- Table-driven tests for multiple scenarios
- Use `t.Fatalf()` for setup failures, `t.Errorf()` for assertion failures
- Run benchmarks: `go test -bench=. ./...`
- Use `t.Helper()` in helper functions to improve error messages
- Mock external dependencies using interfaces

**Testing Tools:**
- `testify`: Assertions library `github.com/stretchr/testify/assert`
- `mockery`: Mock generation `github.com/vektra/mockery`
- `golangci-lint`: Includes test linting

**Coverage Target:**
- Minimum: 80%
- Target: 90% for critical paths (file I/O, data persistence)
- Exclude: main package, error cases in main only

---

### Rust Testing

**Framework**: Built-in `#[test]` attribute with `cargo test`

**Setup:**
- Unit tests: Same file as code, `#[cfg(test)] mod tests { ... }`
- Integration tests: `tests/` directory at crate root
- Run tests: `cargo test`
- Coverage: `cargo tarpaulin` (external tool)
- Example generation: `cargo test --doc` (documentation tests)

**Test Structure:**
```rust
#[cfg(test)]
mod tests {
    use super::*;
    use assert_matches::assert_matches;

    #[test]
    fn test_function_name() {
        // Arrange
        let input = "test";
        let expected = "output";

        // Act
        let result = my_function(input);

        // Assert
        assert_eq!(result, expected);
    }

    #[test]
    fn test_function_error_handling() {
        // Test error cases
        let result = my_function("");
        assert!(result.is_err());
        assert_matches!(result, Err(MyError::EmptyInput));
    }

    #[test]
    #[should_panic(expected = "panic message")]
    fn test_function_panics() {
        my_function("invalid");
    }
}

// Documentation test
/// Opens a file and reads its contents.
///
/// # Examples
/// ```
/// let content = read_file("test.txt")?;
/// assert_eq!(content, "hello");
/// # Ok::<(), std::io::Error>(())
/// ```
pub fn read_file(path: &str) -> Result<String, std::io::Error> {
    todo!()
}
```

**Integration Test (tests/integration_test.rs):**
```rust
use notepad::*;

#[test]
fn test_full_workflow() {
    // Create editor
    let mut editor = Editor::new();

    // Open, edit, save
    editor.open("test.txt").unwrap();
    editor.insert("hello");
    editor.save().unwrap();

    // Verify
    let content = std::fs::read_to_string("test.txt").unwrap();
    assert_eq!(content, "hello");
}
```

**Best Practices:**
- Unit tests in `#[cfg(test)]` modules within implementation files
- Integration tests in `tests/` directory for workflow testing
- Use `assert_eq!` for specific comparisons
- Use custom assertion functions for complex conditions
- Test both `Ok` and `Err` paths for `Result` types
- Use `should_panic` sparingly (prefer `Result<T>` assertions)

**Testing Tools:**
- `proptest`: Property-based testing
- `criterion`: Benchmarking (more sophisticated than built-in)
- `assert_matches`: Better error matching
- `mockall`: Mocking library

**Coverage Target:**
- Minimum: 80%
- Target: 90% for critical paths (file I/O, parsing, validation)
- Exclude: example code, simple getter methods

---

### C Testing

**Framework**: Custom test runner using Check library or simple assert macros

**Setup:**
- Test files: `tests/test_*.c` files
- Build system: CMake with CTest
- Coverage: gcov/lcov
- Run tests: `cmake --build . --target test` or `ctest`

**Test Structure with Assertions:**
```c
#include <stdio.h>
#include <string.h>
#include <assert.h>
#include "../src/module.h"

// Helper macro for cleaner assertions
#define ASSERT_INT_EQ(actual, expected) \
    do { \
        if ((actual) != (expected)) { \
            fprintf(stderr, "FAIL: %s:%d - got %d, expected %d\n", \
                    __FILE__, __LINE__, actual, expected); \
            return 1; \
        } \
    } while(0)

#define ASSERT_STR_EQ(actual, expected) \
    do { \
        if (strcmp((actual), (expected)) != 0) { \
            fprintf(stderr, "FAIL: %s:%d - got %s, expected %s\n", \
                    __FILE__, __LINE__, actual, expected); \
            return 1; \
        } \
    } while(0)

// Unit test
int test_function_name(void) {
    // Arrange
    char input[256] = "test";
    char expected[256] = "output";
    char actual[256];

    // Act
    int result = my_function(input, actual, sizeof(actual));

    // Assert
    ASSERT_INT_EQ(result, 0);  // success code
    ASSERT_STR_EQ(actual, expected);

    return 0;  // test passed
}

// Error handling test
int test_function_error(void) {
    // Test with invalid input
    int result = my_function(NULL, NULL, 0);
    ASSERT_INT_EQ(result, -1);  // error code
    return 0;
}

// Test runner
int main(void) {
    int tests_run = 0;
    int tests_failed = 0;

    // Run tests
    if (test_function_name() != 0) {
        printf("FAIL: test_function_name\n");
        tests_failed++;
    } else {
        printf("PASS: test_function_name\n");
    }
    tests_run++;

    if (test_function_error() != 0) {
        printf("FAIL: test_function_error\n");
        tests_failed++;
    } else {
        printf("PASS: test_function_error\n");
    }
    tests_run++;

    // Summary
    printf("\nRan %d tests. %d failed.\n", tests_run, tests_failed);
    return tests_failed > 0 ? 1 : 0;
}
```

**Alternative: Using Check Library:**
```c
#include <check.h>
#include "../src/module.h"

START_TEST(test_function_name)
{
    char input[] = "test";
    char expected[] = "output";
    char actual[256];

    int result = my_function(input, actual, sizeof(actual));

    ck_assert_int_eq(result, 0);
    ck_assert_str_eq(actual, expected);
}
END_TEST

Suite* module_suite(void)
{
    Suite *s;
    TCase *tc;

    s = suite_create("Module");
    tc = tcase_create("Core");

    tcase_add_test(tc, test_function_name);
    suite_add_tcase(s, tc);

    return s;
}

int main(void)
{
    Suite *s;
    SRunner *sr;
    int number_failed;

    s = module_suite();
    sr = srunner_create(s);

    srunner_run_all(sr, CK_VERBOSE);
    number_failed = srunner_ntests_failed(sr);
    srunner_free(sr);

    return (number_failed == 0) ? EXIT_SUCCESS : EXIT_FAILURE;
}
```

**CMakeLists.txt Setup:**
```cmake
# Enable testing
enable_testing()

# Add test executable
add_executable(test_module tests/test_module.c src/module.c)
target_link_libraries(test_module check m)

# Register test
add_test(NAME module_tests COMMAND test_module)

# Code coverage
if(CMAKE_C_COMPILER_ID MATCHES "GNU")
    target_compile_options(test_module PRIVATE --coverage)
    target_link_libraries(test_module PRIVATE gcov)
endif()
```

**Best Practices:**
- One test function per behavior
- Use helper macros for assertions with file/line info
- Test both success and error paths explicitly
- Use static functions for file-scoped helpers
- Clean up allocated resources with `teardown()` if needed
- Memory leak detection with valgrind: `valgrind ./test_module`

**Testing Tools:**
- `Check`: Unit testing framework with CUnit-like API
- `CMock`: Mock generation and assertion library
- `Valgrind`: Memory error detection
- `gcov/lcov`: Code coverage analysis

**Coverage Target:**
- Minimum: 80%
- Target: 90% for critical paths (file I/O, buffer management)
- Exclude: error logging functions, simple wrapper functions

---

## Test Organization & Naming

### Go
- File location: Same directory as code
- File naming: `*_test.go`
- Function naming: `TestFunctionName` or `TestFunctionName_Scenario`
- Table entry naming: Descriptive strings ("valid input", "empty string", "nil pointer")

### Rust
- File location: Same file (`#[cfg(test)]` module) or `tests/` for integration
- Module naming: `mod tests { ... }`
- Function naming: `test_function_name` or `test_function_name_scenario`
- Case naming: Descriptive in `#[test]` attributes

### C
- File location: `tests/` directory
- File naming: `test_*.c`
- Function naming: `test_function_name` (returns 0=pass, 1=fail)
- Macro naming: `ASSERT_TYPE_OP` (e.g., `ASSERT_INT_EQ`, `ASSERT_PTR_NULL`)

---

## Running Tests in CI/CD

All three language implementations have GitHub Actions workflows that automatically run tests on push and pull requests.

**Execution:**
- Unit tests run first (fastest feedback)
- Integration tests run second
- End-to-end tests run last (if time permits)
- Coverage reports generated and published

**Requirements:**
- All tests must pass before merge to `develop`
- Coverage must not decrease below existing baseline
- No test warnings or deprecated patterns

---

## Performance Testing

### Go
```go
func BenchmarkReadFile(b *testing.B) {
    content := strings.Repeat("hello world\n", 1000)

    b.ResetTimer()  // don't count setup time
    for i := 0; i < b.N; i++ {
        readFile(content)
    }
}

// Run: go test -bench=. -benchmem ./...
```

### Rust
```rust
#[cfg(test)]
mod benches {
    use criterion::{black_box, criterion_group, criterion_main, Criterion};
    use notepad::*;

    fn bench_read_file(c: &mut Criterion) {
        c.bench_function("read_file_1mb", |b| {
            b.iter(|| read_file(black_box("large_file.txt")))
        });
    }

    criterion_group!(benches, bench_read_file);
    criterion_main!(benches);
}

// Run: cargo bench
```

### C
Use custom timing macros or integrate a micro-benchmarking library:
```c
#include <time.h>

#define BENCH_START() clock_t start = clock()
#define BENCH_END(msg) \
    do { \
        clock_t end = clock(); \
        double elapsed = (double)(end - start) / CLOCKS_PER_SEC; \
        printf("%s: %.3f ms\n", msg, elapsed * 1000); \
    } while(0)
```

**Performance Benchmarks:**
- File read (1MB): < 50ms
- File write (1MB): < 100ms
- Editor initialization: < 10ms
- Search operation: < 100ms for typical files

---

## Test Checklist

Before committing tests:

- [ ] **Test names are descriptive**: Clear what is being tested and why
- [ ] **Arrange-Act-Assert pattern**: Each test clearly separated into three phases
- [ ] **One assertion per test**: Or related assertions that test the same behavior
- [ ] **No interdependencies**: Tests can run in any order
- [ ] **Isolation**: No test depends on files, network, or other tests
- [ ] **Error paths tested**: Both success and failure cases covered
- [ ] **Edge cases covered**: Empty input, null pointers, boundary conditions
- [ ] **Performance acceptable**: Tests complete in reasonable time
- [ ] **Coverage adequate**: At least 80% line coverage
- [ ] **Documentation present**: Complex test logic has comments explaining intent

---

## Coverage Analysis

### Measuring Coverage

**Go:**
```bash
go test -cover ./...
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

**Rust:**
```bash
cargo tarpaulin --out Html --output-dir coverage
# Or use llvm-cov for more detailed reports
cargo llvm-cov --html
```

**C:**
```bash
cmake --build . --target coverage  # if CMake target configured
# Or manually with gcov
gcov src/module.c --directory=build/CMakeFiles
lcov --capture --directory build --output-file coverage.info
genhtml coverage.info --output-directory coverage
```

### Coverage Goals
- **Lines covered**: 80% minimum, 90% target
- **Branches covered**: 75% minimum (if/else paths)
- **Functions called**: 85% minimum
- **Critical sections**: 95%+ (file I/O, data persistence, error handling)

### Files to Exclude from Coverage
- `main.c/.go/.rs`: Program entry points
- `version.go`: Version information
- Generated code
- Test files themselves
- Example/documentation code

# Testing Strategy

This document outlines the testing approach, coverage requirements, and best practices for the Tri-Language Notepad project.

---

## Testing Philosophy

1. **Test-Driven Development**: Write tests first, then implementation
2. **Comprehensive Coverage**: >80% code coverage minimum
3. **Multiple Levels**: Unit, integration, and end-to-end tests
4. **Clarity**: Tests should be clear and maintainable
5. **Speed**: Tests should run quickly for fast feedback

---

## Testing Pyramid

```
           ╱╲
          ╱  ╲  End-to-End Tests
         ╱────╲  (~10% of tests)
        ╱      ╲
       ╱────────╲
      ╱  Integration╲  Integration Tests
     ╱    Tests     ╲  (~30% of tests)
    ╱────────────────╲
   ╱                  ╲
  ╱──────────────────── ╲  Unit Tests
 ╱        Unit Tests      ╲ (~60% of tests)
╱──────────────────────────╲
```

---

## Coverage Requirements by Language

### Go
- **Target Coverage**: >80%
- **Critical Paths**: 100% coverage required
- **Tool**: `go test -cover`
- **Report**: Coverage reported in CI/CD

```bash
# Run tests with coverage
go test -v -cover ./...

# Generate coverage profile
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

### Rust
- **Target Coverage**: >80%
- **Critical Paths**: 100% coverage required
- **Tool**: `cargo tarpaulin` or `llvm-cov`
- **Report**: Coverage reported in CI/CD

```bash
# Run tests
cargo test

# Generate coverage report
cargo tarpaulin --out Html --output-dir coverage
```

### C
- **Target Coverage**: >75% (harder in C)
- **Critical Paths**: 90%+ coverage
- **Tool**: `gcov` or `lcov`
- **Report**: Coverage reported in CI/CD

```bash
# Build with coverage flags
gcc -fprofile-arcs -ftest-coverage -o test test.c
./test
gcov test.c
```

---

## Unit Testing

Unit tests focus on individual functions/methods in isolation.

### Go Example

```go
package editor

import (
    "testing"
    "github.com/stretchr/testify/assert"
)

// Test successful insertion
func TestInsert_ValidText(t *testing.T) {
    // Arrange
    editor := NewEditor()
    
    // Act
    err := editor.Insert(0, "Hello")
    
    // Assert
    assert.NoError(t, err)
    assert.Equal(t, "Hello", editor.Content())
}

// Test empty text handling
func TestInsert_EmptyText(t *testing.T) {
    editor := NewEditor()
    err := editor.Insert(0, "")
    
    assert.NoError(t, err)
    assert.Equal(t, "", editor.Content())
}

// Test out of bounds
func TestInsert_OutOfBounds(t *testing.T) {
    editor := NewEditor()
    editor.Insert(0, "Hello")
    
    err := editor.Insert(100, "World")
    
    assert.Error(t, err)
}

// Test multiple insertions
func TestInsert_MultipleInsertions(t *testing.T) {
    editor := NewEditor()
    
    editor.Insert(0, "Hello")
    editor.Insert(5, " ")
    editor.Insert(6, "World")
    
    assert.Equal(t, "Hello World", editor.Content())
}
```

### Rust Example

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_insert_valid_text() {
        // Arrange
        let mut editor = Editor::new();
        
        // Act
        let result = editor.insert(0, "Hello");
        
        // Assert
        assert!(result.is_ok());
        assert_eq!(editor.content(), "Hello");
    }

    #[test]
    fn test_insert_empty_text() {
        let mut editor = Editor::new();
        let result = editor.insert(0, "");
        
        assert!(result.is_ok());
        assert_eq!(editor.content(), "");
    }

    #[test]
    fn test_insert_out_of_bounds() {
        let mut editor = Editor::new();
        editor.insert(0, "Hello").ok();
        
        let result = editor.insert(100, "World");
        
        assert!(result.is_err());
    }

    #[test]
    fn test_multiple_insertions() {
        let mut editor = Editor::new();
        
        editor.insert(0, "Hello").ok();
        editor.insert(5, " ").ok();
        editor.insert(6, "World").ok();
        
        assert_eq!(editor.content(), "Hello World");
    }
}
```

### C Example

```c
#include <assert.h>
#include "editor.h"

void test_insert_valid_text() {
    // Arrange
    Editor *editor = editor_create();
    
    // Act
    int result = editor_insert(editor, 0, "Hello");
    
    // Assert
    assert(result == 0);
    assert(strcmp(editor->content, "Hello") == 0);
    
    editor_destroy(editor);
}

void test_insert_empty_text() {
    Editor *editor = editor_create();
    int result = editor_insert(editor, 0, "");
    
    assert(result == 0);
    assert(strlen(editor->content) == 0);
    
    editor_destroy(editor);
}

void test_insert_out_of_bounds() {
    Editor *editor = editor_create();
    editor_insert(editor, 0, "Hello");
    
    int result = editor_insert(editor, 100, "World");
    
    assert(result == -1); // Error code
    
    editor_destroy(editor);
}
```

---

## Integration Testing

Integration tests verify interactions between modules.

### Go Example

```go
// Test file I/O with editor
func TestFileIntegration_CreateAndEdit(t *testing.T) {
    // Arrange
    editor := NewEditor()
    fileMgr := NewFileManager()
    filename := "test.txt"
    
    // Act
    editor.Insert(0, "Initial content")
    err := fileMgr.SaveFile(filename, editor.Content())
    
    // Assert
    assert.NoError(t, err)
    
    // Read back and verify
    content, err := fileMgr.OpenFile(filename)
    assert.NoError(t, err)
    assert.Equal(t, "Initial content", content)
    
    // Cleanup
    fileMgr.DeleteFile(filename)
}
```

### Rust Example

```rust
#[test]
fn test_file_integration_create_and_edit() {
    // Arrange
    let mut editor = Editor::new();
    let file_mgr = FileManager::new();
    let filename = "test.txt";
    
    // Act
    editor.insert(0, "Initial content").ok();
    let save_result = file_mgr.save_file(filename, &editor.content());
    
    // Assert
    assert!(save_result.is_ok());
    
    // Read back and verify
    let content = file_mgr.open_file(filename).unwrap();
    assert_eq!(content, "Initial content");
    
    // Cleanup
    let _ = file_mgr.delete_file(filename);
}
```

---

## Test Organization

### File Structure

**Go**:
```
internal/editor/
├── editor.go
└── editor_test.go          # Tests in same package
```

**Rust**:
```
src/
├── editor.rs
├── lib.rs
└── tests/
    └── integration_test.rs # Integration tests
```

**C**:
```
src/
├── editor.c
└── editor.h
tests/
└── test_editor.c
```

### Test Naming Convention

**Go**:
```go
func TestFunctionName(t *testing.T) { }
func TestFunctionName_Scenario(t *testing.T) { }
func BenchmarkFunctionName(b *testing.B) { }
```

**Rust**:
```rust
#[test]
fn test_function_name() { }

#[test]
fn test_function_name_scenario() { }
```

**C**:
```c
void test_function_name() { }
void test_function_name_scenario() { }
```

---

## Test Categories

### 1. Happy Path Tests
Test the normal, expected behavior:
```
✓ Insert valid text
✓ Delete valid range
✓ Save valid file
```

### 2. Edge Cases
Test boundary conditions:
```
✓ Insert at position 0
✓ Insert at end of content
✓ Delete entire content
✓ Empty input
```

### 3. Error Cases
Test error handling:
```
✓ Invalid position (out of bounds)
✓ Null/nil input
✓ File not found
✓ Permission denied
✓ Disk full
```

### 4. Performance Tests
Benchmark critical operations:
```
✓ Insert 1000 characters
✓ Find in 100KB file
✓ Save large file
```

---

## Performance Testing

### Go Benchmarks

```go
func BenchmarkInsert(b *testing.B) {
    editor := NewEditor()
    b.ResetTimer()
    
    for i := 0; i < b.N; i++ {
        editor.Insert(0, "test")
    }
}

func BenchmarkFind_LargeContent(b *testing.B) {
    editor := NewEditor()
    // Setup large content
    for i := 0; i < 10000; i++ {
        editor.Insert(i, "line\n")
    }
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        editor.Find("test")
    }
}
```

Run with:
```bash
go test -bench=. -benchmem ./...
```

### Rust Benchmarks

Using Criterion:

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn editor_insert_benchmark(c: &mut Criterion) {
    c.bench_function("insert 100 chars", |b| {
        b.iter(|| {
            let mut editor = Editor::new();
            editor.insert(0, black_box("x".repeat(100).as_str()))
        })
    });
}

criterion_group!(benches, editor_insert_benchmark);
criterion_main!(benches);
```

Run with:
```bash
cargo bench
```

---

## Test Utilities

### Go Testing Patterns

```go
// Table-driven tests
func TestInsert_TableDriven(t *testing.T) {
    tests := []struct {
        name      string
        pos       int
        text      string
        wantErr   bool
        expected  string
    }{
        {"valid", 0, "Hello", false, "Hello"},
        {"empty", 0, "", false, ""},
        {"out of bounds", 100, "Hi", true, ""},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            editor := NewEditor()
            err := editor.Insert(tt.pos, tt.text)
            
            if (err != nil) != tt.wantErr {
                t.Errorf("wantErr = %v, got = %v", tt.wantErr, err != nil)
            }
            if !tt.wantErr && editor.Content() != tt.expected {
                t.Errorf("content = %q, want %q", editor.Content(), tt.expected)
            }
        })
    }
}
```

### Rust Testing Patterns

```rust
// Parameterized tests
#[test]
fn test_insert_various() {
    let cases = vec![
        (0, "Hello", true, "Hello"),
        (0, "", true, ""),
        (100, "Hi", false, ""),
    ];
    
    for (pos, text, should_succeed, expected) in cases {
        let mut editor = Editor::new();
        let result = editor.insert(pos, text);
        
        if should_succeed {
            assert!(result.is_ok());
            assert_eq!(editor.content(), expected);
        } else {
            assert!(result.is_err());
        }
    }
}
```

---

## Continuous Integration Testing

All tests run automatically in CI/CD:

1. **Every commit**: Linting + unit tests
2. **Before merge**: Full test suite + coverage check
3. **On merge to main**: Build + deploy to production

See `.github/workflows/` for CI/CD configuration.

---

## Test Metrics Dashboard

Track over time:
- Code coverage percentage
- Number of tests
- Test execution time
- Flaky test identification
- Performance trends

---

## Troubleshooting Tests

### Flaky Tests
- Tests that pass sometimes, fail other times
- Usually due to: race conditions, timing, randomness
- Solution: Use mocks/stubs, synchronize properly, use fixed seeds

### Slow Tests
- Tests taking >1 second
- Solution: Mock external dependencies, reduce data size, use test-specific configs

### Hard to Test Code
- Tight coupling, global state, hidden dependencies
- Solution: Refactor for testability, use dependency injection

---

## Checklist for Test Review

- [ ] Test covers happy path
- [ ] Test covers edge cases
- [ ] Test covers error conditions
- [ ] Test is isolated (no dependencies on other tests)
- [ ] Test has clear arrange-act-assert structure
- [ ] Test name clearly describes what it tests
- [ ] Coverage targets are met
- [ ] Performance is acceptable

---

**Last Updated**: December 11, 2025

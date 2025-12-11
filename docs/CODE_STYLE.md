# Code Style Guide

This document defines the coding standards and conventions for the Notepad project across all three language implementations (Go, Rust, C).

## General Principles

1. **Readability First**: Code is read far more often than it's written. Optimize for clarity over cleverness.
2. **Consistency**: Follow the established conventions within each language. Consistency is more important than perfection.
3. **Self-Documenting**: Code should be understandable without extensive comments. Use clear variable/function names.
4. **Performance**: Don't optimize prematurely, but write code that's inherently efficient. Use language idioms.

## Naming Conventions

### Variables and Constants

**Go:**
- Variables: `camelCase` (e.g., `bufferSize`, `isActive`)
- Constants: `UPPERCASE_SNAKE_CASE` (e.g., `MAX_BUFFER_SIZE`, `DEFAULT_TIMEOUT`)
- Global exported: `PascalCase` (e.g., `ConfigPath`, `ErrorHandler`)
- Unexported: `camelCase` starting with lowercase

**Rust:**
- Variables: `snake_case` (e.g., `buffer_size`, `is_active`)
- Constants: `UPPERCASE_SNAKE_CASE` (e.g., `MAX_BUFFER_SIZE`)
- Type names: `PascalCase` (structs, enums, traits)
- Module names: `snake_case`
- Enum variants: `PascalCase`

**C:**
- Variables: `snake_case` (e.g., `buffer_size`, `is_active`)
- Constants: `UPPERCASE_SNAKE_CASE` (e.g., `MAX_BUFFER_SIZE`)
- Type names (typedef): `PascalCase` or `snake_case_t` (e.g., `FileBuffer_t`)
- Function names: `snake_case` with module prefix (e.g., `buffer_init`, `buffer_read`)
- Macro names: `UPPERCASE_SNAKE_CASE`

### Function/Method Names

**Go:**
- Exported: `PascalCase` (e.g., `OpenFile`, `ReadBuffer`)
- Unexported: `camelCase` (e.g., `openFile`, `readBuffer`)
- Getter methods: `Name()` not `GetName()`
- Setter methods: `SetName()` format

**Rust:**
- Methods/functions: `snake_case` (e.g., `open_file`, `read_buffer`)
- Trait methods: `snake_case`
- Factory functions: `new()`, `with_capacity()`, etc.

**C:**
- Format: `module_action` (e.g., `file_open`, `buffer_read`)
- Init functions: `module_init`
- Cleanup functions: `module_cleanup` or `module_free`

### Struct/Type Names

All languages use **PascalCase** for type definitions:
- Go: `type FileBuffer struct { ... }`
- Rust: `struct FileBuffer { ... }`
- C: `typedef struct { ... } FileBuffer_t;`

## Line Length

- **Hard limit**: 120 characters
- **Soft limit**: 100 characters (preferred for readability)
- **Exception**: Long URLs, error messages, or imports that cannot be broken
- **Rationale**: Fits standard terminal widths, works on 13" laptops, readable without horizontal scrolling

## Indentation

- **Go**: Tabs (enforced by `gofmt`)
- **Rust**: 4 spaces (enforced by `rustfmt`)
- **C**: 4 spaces (enforced by `clang-format`)
- **All files**: LF line endings (enforced by pre-commit hooks)

## Commenting

### Comment Types

1. **Package/Module Comments** (appear at top of file):
   - One-line summary describing the file's purpose
   - Should explain what the package/module does, not how
   - Example: `// Package buffer provides buffered file I/O operations.`

2. **Function/Method Comments** (appear before declaration):
   - Required for all public functions
   - Format: `// FunctionName [parameters] returns [results], describing behavior.`
   - Should explain the *what* and *why*, not the *how*
   - Include error conditions and edge cases

3. **Inline Comments** (within code blocks):
   - Explain *why* code does something, not *what*
   - Keep to one or two lines
   - Don't state the obvious (`x++  // increment x`)

4. **Type Comments** (before struct/interface/type definitions):
   - Brief description of what the type represents
   - List invariants if non-obvious

### Bad vs. Good Comments

**Bad:**
```
x := 0  // set x to 0
i++     // increment i
if err != nil {  // check if error exists
    return err
}
```

**Good:**
```
// Initialize attempt counter for connection retries
attempts := 0

// Retry loop exits when connection succeeds or max attempts reached
for attempts < maxRetries {
    attempts++
}

// Return error immediately; caller must log and clean up resources
if err != nil {
    return err
}
```

## Block Structure

- One blank line between function/method definitions
- One blank line between logical sections within functions
- No blank lines at start or end of blocks
- Group related declarations together

## Imports

- **Go**: Use `goimports` to organize (standard library, third-party, local)
- **Rust**: Organized by type (self, crate, external)
- **C**: System includes in `<>`, local includes in `""`
- Keep imports sorted alphabetically within each group

---

## Language-Specific Conventions

### Go Style

**File Organization:**
```
1. Package declaration
2. Comment describing package purpose (if needed)
3. Import declarations
4. Type declarations
5. Constant declarations
6. Variable declarations
7. Function declarations (exported first, then unexported)
```

**Naming Conventions:**
- Interfaces: `Reader`, `Writer`, `Handler` (often one-word or Agent noun)
- Getter: `Name()` not `GetName()`
- Setter: `SetName()` for mutable receivers
- Receiver variables: Use two-letter abbreviation (`b` for buffer, `f` for file)

**Error Handling:**
- Always check `err != nil` immediately after operation
- Use sentinel errors for expected failures: `var ErrNotFound = errors.New("not found")`
- Wrap errors with context: `fmt.Errorf("failed to read config: %w", err)`
- Don't create unnecessary error types; use concrete types when context matters

**Function Length:**
- Prefer small functions (< 50 lines)
- If > 100 lines, consider splitting into smaller functions
- Exception: Generated code or very specific algorithms

**Defer Usage:**
- Use `defer` for resource cleanup (files, locks, transactions)
- Place defer immediately after acquiring resource
- Order matters: last deferred executes first (LIFO)

**Interfaces:**
- Keep interfaces small (1-3 methods)
- Use interface{} sparingly; prefer concrete types or constraints (Go 1.18+)
- Name interfaces with `-er` suffix when possible

### Rust Style

**File Organization:**
```
1. Module declarations
2. Use statements (imports)
3. Type definitions (structs, enums, traits)
4. Trait implementations
5. Function implementations
6. Module definitions (if using inline modules)
```

**Naming Conventions:**
- `Self::Associated` for associated functions and constants
- Use type aliases for commonly repeated types
- Test modules: `#[cfg(test)] mod tests { ... }`

**Error Handling:**
- Prefer `Result<T, E>` over panics in library code
- Use `?` operator for error propagation
- Create custom error types with `thiserror` or `anyhow`
- Example: `pub fn read_file(path: &str) -> Result<String, IoError>`

**Lifetimes:**
- Use explicit lifetimes in public APIs for clarity
- Avoid lifetime elision in complex scenarios
- Document lifetime relationships in comments when non-obvious
- Use `&'static` sparingly; prefer borrowed references

**Borrowing:**
- Prefer `&T` over `Box<T>` for parameters
- Use `&mut T` only when mutation is necessary
- Avoid `clone()` unless semantically appropriate
- Implement `Copy` for small value types

**Pattern Matching:**
- Use `match` for exhaustive checking
- Use `if let` for single pattern cases
- Avoid nested patterns; extract into helper functions
- Match on references: `match &value { ... }` to avoid moves

**Testing:**
- Tests in same file, `#[cfg(test)]` module at bottom
- Use descriptive test names: `test_parse_valid_config`, not `test1`
- Use `assert_eq!`, `assert!` for clarity
- Test both success and failure paths

### C Style

**File Organization:**
```
1. Include guards (or #pragma once)
2. System includes (<stdio.h>, etc.)
3. Local includes ("module.h", etc.)
4. Type definitions and macros
5. Forward declarations
6. Function declarations
7. Function implementations (in .c files)
```

**Naming Conventions:**
- Function prefix with module name: `buffer_init`, `file_open`
- Static functions (file-scope): `static int process_data(...)`
- Typedef structs: `typedef struct { ... } Name_t;`
- Macros: `UPPERCASE_SNAKE_CASE`

**Memory Management:**
- Always pair `malloc` with `free`
- Initialize pointers: `int *ptr = NULL;` (not uninitialized)
- Check allocation: `if (ptr == NULL) { return NULL; }`
- Use `calloc` when zero-initialization needed
- Document ownership: who allocates, who frees?
- Consider fixed-size buffers over dynamic allocation when bounds known

**Error Handling:**
- Return error codes (not exceptions): `0` for success, non-zero for errors
- Use `errno` for system call failures
- Define error codes: `#define ERR_OPEN_FAILED -1`
- Early returns: `if (condition) { cleanup(); return error; }`

**Function Length:**
- Keep functions under 75 lines
- If > 100 lines, split into smaller functions
- Helper functions can be static in same file

**Pointer Usage:**
- Declare with type: `int *ptr;` not `int* ptr;`
- Check for NULL after allocation and dereferencing
- Use `const` pointers when appropriate: `const char *str`
- Document pointer lifetime in comments

**Macros:**
- Use `#define` for constants only when necessary
- Prefer `enum` for related constants
- Wrap arguments in parentheses: `#define MAX(a,b) ((a) > (b) ? (a) : (b))`
- Document purpose and usage

---

## Complexity Limits

### Cyclomatic Complexity
- **Soft limit**: 10 per function
- **Hard limit**: 20 per function (should be refactored)
- Measured as number of independent paths through code
- Tools: `gocyclo` (Go), `mccabe` (Rust/C)

### Function Length
- **Go**: < 50 lines preferred, max 100 lines
- **Rust**: < 50 lines preferred, max 100 lines
- **C**: < 75 lines preferred, max 100 lines
- Helpers and one-liners exempt from length limits

### File Size
- **Go**: < 500 lines preferred, max 1000 lines
- **Rust**: < 400 lines preferred, max 800 lines
- **C Header**: < 200 lines preferred, max 300 lines
- **C Implementation**: < 500 lines preferred, max 1000 lines

### Nesting Depth
- **Hard limit**: 4 levels
- **Soft limit**: 3 levels
- Use early returns to reduce nesting
- Extract nested logic into helper functions

---

## Testing Style

### Test Naming
- **Go**: `TestFunctionName`, `TestFunctionName_ScenarioDescription`
- **Rust**: `#[test] fn test_function_name() { ... }`
- **C**: `void test_function_name(void) { ... }`

### Test Organization
- One test per behavior/condition
- Arrange-Act-Assert (AAA) pattern:
  ```
  // Arrange: set up test data
  // Act: execute function
  // Assert: verify results
  ```

### Test Coverage
- Minimum: 80% code coverage
- Target: 90% for critical modules
- Exclude: boilerplate, error cases in main/init
- Tools: Go `testing` package, Rust `cargo tarpaulin`, C `gcov`

### Table-Driven Tests
**Go:**
```go
tests := []struct {
    name    string
    input   string
    want    string
    wantErr bool
}{
    {"valid input", "test", "result", false},
    {"empty input", "", "", true},
}
for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        // test logic
    })
}
```

**Rust:**
```rust
#[test]
fn test_cases() {
    let cases = vec![
        ("input", "output"),
    ];
    for (input, expected) in cases {
        assert_eq!(process(input), expected);
    }
}
```

### Assertion Style
- Be specific: `assert_eq!(actual, expected)` not `assert!(x == y)`
- Include messages for complex assertions
- One assertion per test when possible

---

## Documentation Requirements

### Required Documentation
1. **All public functions/methods**: Must have documentation comment
2. **All exported types**: Must have documentation comment
3. **All modules/packages**: Must have documentation comment
4. **Complex algorithms**: Explain approach and time/space complexity
5. **Edge cases**: Document limitations and error conditions

### Documentation Format

**Go (doc comments):**
```go
// Buffer provides buffered read/write operations on files.
type Buffer struct { ... }

// Read returns the next n bytes from the buffer.
// If EOF is reached, returns data read and io.EOF.
func (b *Buffer) Read(n int) ([]byte, error) { ... }
```

**Rust (doc comments):**
```rust
/// Buffer provides buffered read/write operations on files.
pub struct Buffer { ... }

/// Returns the next n bytes from the buffer.
///
/// # Errors
/// Returns `IoError` if the underlying file cannot be read.
pub fn read(&mut self, n: usize) -> Result<Vec<u8>, IoError> { ... }
```

**C (header comments):**
```c
/**
 * Buffer provides buffered read/write operations on files.
 */
typedef struct { ... } Buffer_t;

/**
 * Reads up to n bytes from the buffer.
 * @param buf The buffer to read from
 * @param n Number of bytes to read
 * @return Number of bytes read, or -1 on error
 */
int buffer_read(Buffer_t *buf, size_t n);
```

---

## Code Review Checklist

Before submitting code for review, verify:

- [ ] **Naming**: All variables, functions, types follow conventions
- [ ] **Line Length**: No lines exceed 120 characters
- [ ] **Indentation**: Consistent with language standard (tabs/spaces/LF)
- [ ] **Comments**: Public APIs documented, inline comments explain *why*
- [ ] **Complexity**: Cyclomatic complexity < 10, functions < 100 lines
- [ ] **Error Handling**: All error paths covered and tested
- [ ] **Testing**: Tests written, 80%+ coverage achieved
- [ ] **Formatting**: Runs through language formatter without changes
- [ ] **Imports**: Organized and no unused imports
- [ ] **Performance**: No obvious inefficiencies or N² algorithms
- [ ] **Security**: No hardcoded secrets, safe input validation
- [ ] **Documentation**: All public APIs documented with examples

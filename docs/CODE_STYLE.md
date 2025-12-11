# Code Style Guide

This document defines the code style and conventions for the Tri-Language Notepad project.

## General Principles

1. **Readability First**: Code is read more often than written
2. **Consistency**: Keep code consistent across the project
3. **Clarity**: Explicit is better than implicit
4. **Simplicity**: Keep it simple and straightforward
5. **Performance**: Don't sacrifice readability for micro-optimizations

---

## Cross-Language Standards

### Naming Conventions

#### Variables & Constants
- **Variables**: `camelCase` (Go, Rust, C)
  ```
  Good: currentPosition, fileHandle, bufferSize
  Bad: current_position, CurrentPosition, CURRENT_POSITION
  ```

- **Constants**: `UPPER_SNAKE_CASE`
  ```
  Good: MAX_BUFFER_SIZE, DEFAULT_TIMEOUT
  Bad: MaxBufferSize, max_buffer_size
  ```

- **Functions**: `camelCase` (Go, Rust) or `snake_case` (C)
  ```
  Go/Rust: insertText(), deleteContent()
  C: insert_text(), delete_content()
  ```

- **Classes/Structs**: `PascalCase`
  ```
  Good: TextBuffer, FileManager, EditorEngine
  Bad: textBuffer, text_buffer, TEXTBUFFER
  ```

#### Abbreviations
- Avoid cryptic abbreviations
- Use full words when possible
```
Good: currentIndex, documentTitle, fileSystemPath
Bad: curIdx, docTtl, fsp
```

### Line Length

- **Soft limit**: 80 characters (warnings)
- **Hard limit**: 120 characters (must break)
- **Exception**: URLs and long strings can exceed

### Indentation

- **Go**: Tabs (per Go convention)
- **Rust**: 4 spaces
- **C**: 4 spaces
- Use EditorConfig for automatic enforcement

### Comments

#### Comment Placement
```
// Place comments above the code they describe
// Use complete sentences with proper capitalization

function insert(text: String) {
    // Remove leading/trailing whitespace before inserting
    let trimmed = text.trim()
    buffer.append(trimmed)
}
```

#### Good Comments
- Explain **why**, not **what**
- Clarify non-obvious logic
- Reference external sources
- Mark incomplete code with TODO/FIXME

```go
// FIXME: This algorithm is O(n²), optimize for large buffers
// See: https://en.wikipedia.org/wiki/Boyer-Moore_algorithm

// Insert with position validation to prevent out-of-bounds access
editor.Insert(position, text)
```

#### Bad Comments
- Stating the obvious
- Incorrect information
- Obsolete comments

```go
// Bad: This doesn't add value
i++ // Increment i

// Bad: Misleading comment
// This function is very fast
// (Function is actually slow)
```

#### Documentation Comments

**Go**:
```go
// Insert adds text at the specified position in the buffer.
// It returns an error if the position is out of bounds.
// This operation can be undone with the Undo function.
func (e *Editor) Insert(pos int, text string) error {
```

**Rust**:
```rust
/// Inserts text at the specified position in the buffer.
///
/// # Arguments
/// * `pos` - The position where text will be inserted
/// * `text` - The text to insert
///
/// # Returns
/// Returns `Ok(())` on success, `Err(EditorError)` on failure
///
/// # Example
/// ```
/// let mut editor = Editor::new();
/// editor.insert(0, "Hello")?;
/// ```
pub fn insert(&mut self, pos: usize, text: &str) -> Result<(), EditorError> {
```

**C**:
```c
/**
 * Inserts text at the specified position in the buffer.
 *
 * @param editor The editor instance
 * @param pos The position where text will be inserted
 * @param text The text to insert
 * @return 0 on success, -1 on error
 */
int editor_insert(Editor *editor, int pos, const char *text);
```

---

## Go Style Guide

### File Organization
```go
// 1. Package declaration
package editor

// 2. Imports (standard library, then third-party)
import (
    "fmt"
    "io"
    
    "github.com/pkg/errors"
)

// 3. Constants
const DefaultBufferSize = 1024

// 4. Variables
var globalRegistry *Registry

// 5. Interfaces
type Reader interface { }

// 6. Structs
type Editor struct { }

// 7. Methods and functions
func (e *Editor) Insert(text string) error { }

func NewEditor() *Editor { }
```

### Naming
```go
// Exported: Capitalize first letter
type Editor struct { }
func (e *Editor) Insert(text string) error { }

// Unexported: lowercase
type buffer struct { }
func (b *buffer) write(text string) { }
```

### Error Handling
```go
// Good: Check errors immediately
if err != nil {
    return fmt.Errorf("failed to insert text: %w", err)
}

// Good: Wrap errors with context
file, err := os.Open(path)
if err != nil {
    return nil, errors.Wrap(err, "could not open file")
}
```

### Function Length
- Keep functions under 50 lines (aim for 20-30)
- Extract complex logic into helper functions
- One responsibility per function

---

## Rust Style Guide

### File Organization
```rust
// 1. Module declarations
mod editor;

// 2. Imports
use std::fs;
use crate::models::Document;

// 3. Constants
const DEFAULT_BUFFER_SIZE: usize = 1024;

// 4. Structs and Traits
pub struct Editor { }
pub trait Editable { }

// 5. Implementations
impl Editor { }
impl Editable for Editor { }

// 6. Functions
pub fn create_editor() -> Editor { }
```

### Naming
```rust
// Types: PascalCase
pub struct TextBuffer { }
pub enum EditorError { }

// Functions/variables: snake_case
pub fn insert_text(text: &str) { }
let current_position = 0;

// Constants: SCREAMING_SNAKE_CASE
const MAX_BUFFER_SIZE: usize = 1_048_576;
```

### Error Handling
```rust
// Good: Use Result types
pub fn insert(&mut self, text: &str) -> Result<(), EditorError> {
    if text.is_empty() {
        return Err(EditorError::EmptyText);
    }
    Ok(())
}

// Good: Use ? operator
fn process_file(path: &str) -> Result<Content, FileError> {
    let content = std::fs::read_to_string(path)?;
    Ok(content)
}
```

### Lifetimes
```rust
// Explicit when needed
fn find<'a>(haystack: &'a str, needle: &str) -> Option<&'a str>

// Avoid unnecessary lifetime parameters
// Bad: fn insert<'a>(&'a mut self, text: &'a str)
// Good:
fn insert(&mut self, text: &str)
```

---

## C Style Guide

### File Organization

**Header files** (.h):
```c
// 1. Include guards
#ifndef EDITOR_H
#define EDITOR_H

// 2. Includes
#include <stdio.h>
#include "buffer.h"

// 3. Constants
#define MAX_BUFFER_SIZE 1024

// 4. Type definitions
typedef struct {
    char *content;
    int length;
} Editor;

// 5. Function declarations
int editor_insert(Editor *e, const char *text);
int editor_delete(Editor *e, int start, int end);

#endif // EDITOR_H
```

**Implementation files** (.c):
```c
// 1. Includes
#include "editor.h"
#include <stdlib.h>

// 2. Static constants and variables
static const int INITIAL_CAPACITY = 256;

// 3. Static helper functions
static int capacity_for_length(int len) {
    // ...
}

// 4. Public function implementations
int editor_insert(Editor *e, const char *text) {
    // ...
}
```

### Naming
```c
// Functions: snake_case with module prefix
editor_insert()
editor_delete()
file_open()
file_save()

// Structs: snake_case
struct text_buffer { }

// Constants: SCREAMING_SNAKE_CASE
#define MAX_LINE_LENGTH 120
#define DEFAULT_TAB_WIDTH 4
```

### Memory Management
```c
// Good: Check allocation
Editor *e = malloc(sizeof(Editor));
if (e == NULL) {
    return NULL; // or appropriate error
}

// Good: Clean up
free(e->content);
free(e);
e = NULL; // Avoid use-after-free

// Good: Use size safely
int size = strlen(text) + 1;
char *copy = malloc(size);
if (!copy) { /* handle error */ }
strncpy(copy, text, size - 1);
copy[size - 1] = '\0';
```

### Error Handling
```c
// Good: Return error codes
int editor_insert(Editor *e, const char *text) {
    if (e == NULL || text == NULL) {
        return -1; // Invalid argument
    }
    if (strlen(text) > MAX_TEXT_LENGTH) {
        return -2; // Text too long
    }
    // ... implementation
    return 0; // Success
}
```

---

## Formatting & Tools

### Automated Formatting

All code MUST pass automated formatting:

**Go**:
```bash
go fmt ./...
goimports -w .
```

**Rust**:
```bash
cargo fmt
```

**C**:
```bash
clang-format -i src/*.{c,h}
```

### Pre-commit Hooks

Formatting checks run automatically before commits. To run manually:

```bash
pre-commit run --all-files
```

---

## Complexity Limits

### Cyclomatic Complexity
- **Max**: 10 per function
- **Target**: <5 average
- **Tool**: Linters check automatically

### Function Length
- **Max**: 100 lines
- **Target**: 20-50 lines
- Extract complex logic into helpers

### File Size
- **Max**: 500 lines
- **Target**: 200-300 lines
- Split into multiple files when needed

---

## Testing Style

### Test Naming
```go
// Go: TestFunctionName_Scenario
func TestInsert_ValidText(t *testing.T) { }
func TestInsert_EmptyText(t *testing.T) { }
func TestInsert_OutOfBounds(t *testing.T) { }
```

```rust
// Rust: test_function_name_scenario
#[test]
fn test_insert_valid_text() { }
fn test_insert_empty_text() { }
fn test_insert_out_of_bounds() { }
```

### Test Structure (AAA)
```
Arrange: Set up test data
Act: Execute the function
Assert: Verify the results
```

---

## Performance Considerations

1. **Premature Optimization**: Don't optimize without evidence
2. **Benchmarking**: Measure before and after changes
3. **Profiling**: Use language tools to find bottlenecks
4. **Complexity**: Use appropriate algorithms and data structures

---

## Documentation Checklist

- [ ] All public functions have doc comments
- [ ] Complex logic has inline comments explaining "why"
- [ ] Examples provided for non-obvious APIs
- [ ] Parameters and return values documented
- [ ] Error conditions documented
- [ ] Performance characteristics noted (if relevant)

---

**Last Updated**: December 11, 2025

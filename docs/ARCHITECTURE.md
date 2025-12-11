# Architecture Overview

## Project Architecture

This document describes the overall architecture and design decisions for the Tri-Language Notepad project.

### Layered Architecture

The application follows a classic **layered architecture** pattern:

```
┌─────────────────────────────────┐
│         UI Layer                 │
│    (User Interface/CLI)          │
├─────────────────────────────────┤
│      Editor Engine               │
│   (Text Processing Logic)        │
├─────────────────────────────────┤
│     File Manager                 │
│  (File I/O Operations)           │
├─────────────────────────────────┤
│    Configuration System          │
│   (Settings & Preferences)       │
├─────────────────────────────────┤
│       Core Models                │
│   (Data Structures)              │
└─────────────────────────────────┘
```

---

## Component Design

### 1. **Core Models Layer**
Defines the fundamental data structures:

- **Document**: Text content with metadata
- **Buffer**: In-memory text storage with cursor tracking
- **UndoStack**: Undo/redo history management
- **Statistics**: Document statistics (word count, char count)

### 2. **Configuration System**
Manages application settings:

- User preferences (theme, font, auto-save interval)
- Recent files list
- Default file locations
- Application settings persistence

### 3. **File Manager**
Handles all file operations:

- Create, read, write, delete files
- File format detection
- Backup management
- Error handling for file operations

**Key Methods**:
- `CreateFile(path)` - Create new file
- `OpenFile(path)` - Read file content
- `SaveFile(path, content)` - Write file
- `DeleteFile(path)` - Delete file
- `CreateBackup(path)` - Create backup copy

### 4. **Editor Engine**
Core text editing logic:

- Text manipulation (insert, delete, replace)
- Find and replace functionality
- Undo/redo operations
- Line management
- Statistics calculation

**Key Methods**:
- `Insert(position, text)` - Insert text
- `Delete(start, end)` - Delete text range
- `Find(pattern)` - Search for pattern
- `Replace(pattern, replacement)` - Find and replace
- `Undo()` / `Redo()` - Undo/redo operations

### 5. **UI Layer**
User interface (language-specific):

- Command-line interface or GUI
- User input handling
- Output display
- Error messaging

---

## Data Flow

### File Opening
```
User → UI → FileManager → Read File → Buffer → Editor → Display
```

### Text Editing
```
User Input → UI → Editor Engine → Buffer → UndoStack → Update Display
```

### Save Operation
```
Editor → FileManager → Validate Path → Write File → Backup → Confirm
```

---

## Design Patterns Used

### 1. **Model-View-Separation**
- Data (Models) separated from display (UI)
- UI only handles user interaction and display
- Models handle business logic

### 2. **Single Responsibility Principle**
- Each module has one reason to change
- FileManager: only file operations
- Editor: only text editing
- UI: only user interaction

### 3. **Dependency Injection**
- Components receive dependencies
- Easier to test and swap implementations
- Reduces tight coupling

### 4. **Error Handling Pattern**
- All operations return Result types
- Rust: `Result<T, E>`
- Go: `(T, error)`
- C: Return codes with error structs

---

## Language-Specific Implementations

### Go Implementation
- **Concurrency**: Goroutines for async file operations
- **Channels**: Communication between components
- **Interfaces**: Polymorphic behavior
- **Error Handling**: `error` interface

### Rust Implementation
- **Ownership**: Memory safety without GC
- **Traits**: Polymorphic behavior and constraints
- **Result/Option**: Null safety
- **Lifetimes**: Reference validation

### C Implementation
- **Structs**: Data organization
- **Function Pointers**: Polymorphism
- **Manual Memory**: Explicit allocation/deallocation
- **Error Codes**: Return value status

---

## State Management

### Document State
```
┌─────────────────┐
│  File System    │
└────────┬────────┘
         │
    ┌────▼────┐
    │ File I/O │
    └────┬────┘
         │
    ┌────▼──────────┐
    │ Buffer State  │
    │ - Content     │
    │ - Cursor Pos  │
    │ - Metadata    │
    └────┬──────────┘
         │
    ┌────▼──────────┐
    │ Undo/Redo     │
    │ Stack         │
    └────┬──────────┘
         │
    ┌────▼──────────┐
    │ Display       │
    └───────────────┘
```

---

## Performance Considerations

1. **Text Buffer**: Use rope or gap buffer for efficient insertions
2. **Search**: Index-based or incremental search for large files
3. **Undo/Redo**: Limit stack size to prevent memory bloat
4. **File I/O**: Async operations for non-blocking UI
5. **Rendering**: Only update changed lines

---

## Testing Strategy

### Unit Tests
- Test each module independently
- Mock dependencies
- Test error cases

### Integration Tests
- Test module interactions
- Full workflow tests
- File system integration

### Performance Tests
- Benchmark critical operations
- Memory profiling
- Load testing with large files

---

## Security Considerations

1. **Input Validation**: Sanitize all user input
2. **Path Traversal**: Prevent directory traversal attacks
3. **File Permissions**: Respect OS file permissions
4. **Memory Safety**: Prevent buffer overflows (especially C)
5. **Resource Limits**: Prevent DoS with large files

---

## Future Enhancements

### Phase 2 Features
- Multi-document support
- Plugin system
- Advanced syntax highlighting
- Cloud synchronization
- Collaboration features

### Phase 3+ Enhancements
- Version control integration
- Diff/merge capabilities
- Terminal integration
- Debugger integration

---

**Last Updated**: December 11, 2025

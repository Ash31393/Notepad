# Architecture Overview

## Project Architecture

This document describes the overall architecture and design decisions for the Tri-Language Notepad project.

### Layered Architecture

The application follows a classic **layered architecture** pattern:
┌─────────────────────────────────┐
│ UI Layer │
│ (User Interface/CLI) │
├─────────────────────────────────┤
│ Editor Engine │
│ (Text Processing Logic) │
├─────────────────────────────────┤
│ File Manager │
│ (File I/O Operations) │
├─────────────────────────────────┤
│ Configuration System │
│ (Settings & Preferences) │
├─────────────────────────────────┤
│ Core Models │
│ (Data Structures) │
└─────────────────────────────────┘
---

## Component Design

### 1. **Core Models Layer**
Defines the fundamental data structures:
- **Document**: Text content with metadata
- **Buffer**: In-memory text storage with cursor tracking
- **UndoStack**: Undo/redo history management

### 2. **File Manager**
Handles all file operations:
- Create, read, write, delete files
- File format detection
- Backup management

### 3. **Editor Engine**
Core text editing logic:
- Text manipulation (insert, delete, replace)
- Find and replace functionality
- Undo/redo operations

### 4. **Configuration System**
Manages application settings:
- User preferences
- Recent files list
- Default file locations

### 5. **UI Layer**
User interface (language-specific):
- Command-line interface or GUI
- User input handling
- Output display

---

## Data Flow

### File Opening

User → UI → FileManager → Read File → Buffer → Editor → Display

### Text Editing

User Input → UI → Editor Engine → Buffer → UndoStack → Update Display
### Save Operation

Editor → FileManager → Validate Path → Write File → Backup → Confirm


---

## Design Patterns Used

### 1. **Model-View-Separation**
- Data (Models) separated from display (UI)
- UI only handles user interaction and display

### 2. **Single Responsibility Principle**
- Each module has one reason to change
- FileManager: only file operations
- Editor: only text editing
- UI: only user interaction

### 3. **Error Handling Pattern**
- All operations return Result types
- Rust: `Result<T, E>`
- Go: `(T, error)`
- C: Return codes with error structs

---

**Last Updated**: December 11, 2025

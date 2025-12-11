# Contributing Guidelines

Thank you for your interest in contributing to the Tri-Language Notepad project! This document provides guidelines and instructions for contributing.

## Code of Conduct

- Be respectful and professional
- Provide constructive feedback
- Help others learn and grow
- Report issues responsibly

## Getting Started

1. **Fork the repository** on GitHub
2. **Clone your fork** locally
3. **Follow the setup instructions** in DEVELOPMENT.md
4. **Create a feature branch** from `develop`

```bash
git checkout develop
git pull origin develop
git checkout -b feature/your-feature-name
```

## Development Process

### Before You Start

1. Check existing issues to avoid duplicate work
2. Discuss major changes in an issue first
3. Follow the project's style guidelines
4. Set up pre-commit hooks: `pre-commit install`

### Writing Code

1. **Write tests first** (Test-Driven Development)
2. **Keep commits small and logical**
3. **Use conventional commit format**:
   ```
   type(scope): description
   
   - type: feat, fix, docs, style, refactor, perf, test, chore
   - scope: the area affected (editor, file, ui, config)
   - description: what changed and why
   ```

4. **Examples**:
   ```
   feat(editor): add find and replace functionality
   fix(file): handle missing file gracefully
   docs: update setup instructions
   test(editor): add undo/redo tests
   refactor(file): simplify file reading logic
   perf(editor): optimize search algorithm
   ```

### Code Quality Requirements

Before committing:

```bash
# Format code
cd go && go fmt ./... && cd ..
cd rust && cargo fmt && cd ..

# Run linters
cd go && golangci-lint run && cd ..
cd rust && cargo clippy && cd ..

# Run tests
cd go && go test -v -cover ./... && cd ..
cd rust && cargo test && cd ..
cd c/build && make test && cd ../..

# Check with pre-commit
pre-commit run --all-files
```

### Documentation

- **Code Comments**: Explain "why", not "what"
- **Function Documentation**: Document all public APIs
- **Examples**: Provide usage examples for complex functions
- **Update CHANGELOG.md** for user-facing changes

---

## Pull Request Process

### Before Creating PR

1. **Rebase on latest develop**:
   ```bash
   git fetch origin
   git rebase origin/develop
   ```

2. **Push your branch**:
   ```bash
   git push origin feature/your-feature-name
   ```

### Creating the PR

1. Go to GitHub and click "Compare & pull request"
2. **Target**: `develop` branch (not `main`)
3. **Title**: Use conventional commit format
4. **Description**: Use the PR template to fill in details
5. **Link related issues**: Reference issue numbers

### PR Checklist

You must ensure:

- ✅ Tests pass (`CI/CD` shows green)
- ✅ Code coverage >80%
- ✅ Linting passes (100%)
- ✅ Documentation updated
- ✅ No merge conflicts
- ✅ Commits are clean and descriptive

### Code Review

1. **Automated Review**: CodeRabbit will review your code
2. **Human Review**: A maintainer will review
3. **Address Feedback**: 
   - Make requested changes
   - Push new commits (don't force push)
   - Respond to comments

4. **Approval**: After approval, code will be merged

---

## Commit Message Guidelines

### Format
```
type(scope): subject

body

footer
```

### Type
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Code style (formatting, missing semicolons, etc)
- `refactor`: Code refactoring
- `perf`: Performance improvement
- `test`: Adding or updating tests
- `chore`: Build, dependencies, etc

### Scope
- `editor`: Editor engine
- `file`: File management
- `ui`: User interface
- `config`: Configuration system
- `test`: Testing infrastructure
- `docs`: Documentation
- `go`: Go implementation
- `rust`: Rust implementation
- `c`: C implementation

### Subject
- Imperative mood ("add" not "adds" or "added")
- Don't capitalize first letter
- No period at end
- 50 characters or less

### Body
- Explain what and why, not how
- Wrap at 72 characters
- Separate from subject with blank line

### Footer
- Reference issues: `Closes #123`
- Breaking changes: `BREAKING CHANGE: description`

### Examples

```
feat(editor): add find and replace functionality

Implement find and replace with support for:
- Simple text search
- Case-sensitive option
- Replace single or all occurrences

Closes #42
```

```
fix(file): handle file not found error gracefully

Previously, missing files would crash the app.
Now we display a user-friendly error message
and allow the user to create a new file.

Fixes #15
```

---

## Testing Guidelines

### Coverage Requirements
- **Target**: >80% code coverage
- **Critical paths**: 100% coverage
- **Tools**: 
  - Go: `go test -cover`
  - Rust: `cargo tarpaulin`
  - C: `gcov`

### Test Organization

**Go**:
```
internal/editor/
  ├── editor.go
  └── editor_test.go
```

**Rust**:
```
src/
  ├── editor.rs
  └── tests/editor_tests.rs
```

**C**:
```
src/
  ├── editor.c
  └── ../tests/editor_test.c
```

### Test Naming

- **Go**: `TestFunctionName`, `TestFunctionName_EdgeCase`
- **Rust**: `test_function_name`, `test_function_name_edge_case`
- **C**: `test_function_name`, `test_function_name_edge_case`

### Test Structure

```
1. Setup (arrange)
2. Execute (act)
3. Verify (assert)
```

### Example (Go)
```go
func TestInsertText(t *testing.T) {
    // Arrange
    editor := NewEditor()
    
    // Act
    editor.Insert("Hello")
    
    // Assert
    assert.Equal(t, "Hello", editor.Content())
}
```

---

## Code Style Guidelines

### General
- Max line length: 100 characters
- 4 spaces indentation (except Go which uses tabs)
- One blank line between functions/methods
- No trailing whitespace

### Go
- Follow [Effective Go](https://golang.org/doc/effective_go)
- Use `gofmt` for formatting
- Use `golangci-lint` for linting
- Export comments: `// FunctionName does X`

### Rust
- Follow [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
- Use `rustfmt` for formatting
- Use `clippy` for linting
- Doc comments: `/// Describes the function`

### C
- Follow [CERT Coding Standards](https://wiki.sei.cmu.edu/confluence/display/c)
- Use `clang-format` for formatting
- Use `clang-tidy` for linting
- Comments: `/* Describes the function */`

---

## Reporting Bugs

1. **Check existing issues** - Don't duplicate
2. **Create detailed bug report**:
   - Steps to reproduce
   - Expected behavior
   - Actual behavior
   - Screenshots if applicable
   - Environment info (OS, language version)

3. **Use the bug report template** on GitHub

---

## Requesting Features

1. **Check existing issues** first
2. **Create feature request** with:
   - Problem statement (what problem does it solve?)
   - Proposed solution
   - Alternative approaches considered
   - Use cases and benefits

3. **Use the feature request template** on GitHub

---

## Questions & Discussion

- **Discussions**: GitHub Discussions for questions
- **Issues**: GitHub Issues for bugs and features
- **Email**: For security issues only (no public discussion)

---

## Recognition

Contributors will be:
- Listed in CHANGELOG.md
- Credited in documentation
- Added to contributors list

---

## Legal

By contributing, you agree that:
- Your contributions are your own work
- You grant rights for your code to be used under project's license
- You have rights to contribute

---

**Last Updated**: December 11, 2025

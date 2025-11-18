# AI-Assisted Development: Reducing LLM Hallucination

## Overview

This document provides guidelines for developers using Large Language Models (LLMs) and AI assistants when contributing to the Longhorn SPDK Engine project. Following these best practices helps reduce AI hallucinations and ensures high-quality, accurate code contributions.

## Understanding LLM Hallucination

LLM hallucination refers to instances where AI models generate plausible-sounding but incorrect or fabricated information. In software development, this can manifest as:

- Inventing non-existent APIs or functions
- Suggesting incorrect configurations or parameters
- Misunderstanding project architecture or dependencies
- Creating code that compiles but doesn't achieve the intended functionality
- Fabricating documentation or citing non-existent sources

## Best Practices for Reducing Hallucination

### 1. Provide Specific Context

When using AI assistants, always provide:

- **Exact file paths and names** from the repository
- **Relevant code snippets** with surrounding context
- **Version information** for dependencies (from `go.mod`)
- **Specific error messages** when debugging
- **Links to actual documentation** (SPDK, Longhorn, Go libraries)

**Example:**
```
❌ Bad: "How do I use the SPDK library in this project?"
✅ Good: "In pkg/spdk/engine.go, I need to call the SPDK bdev_raid_create 
function. Looking at the existing code around line 1688, how should I 
structure the parameters based on our current RAID setup?"
```

### 2. Verify AI Suggestions Against Source Code

Before implementing AI-generated code:

1. **Check if suggested functions exist** in the codebase
   ```bash
   grep -r "functionName" pkg/
   ```

2. **Verify API signatures** match actual implementations
   ```bash
   go doc package.Type.Method
   ```

3. **Review existing test cases** to understand expected behavior
   ```bash
   find . -name "*_test.go" -exec grep -l "FunctionName" {} \;
   ```

### 3. Cross-Reference with Official Documentation

Always validate AI suggestions against authoritative sources:

- **SPDK Documentation**: https://spdk.io/doc/
- **Longhorn Documentation**: https://longhorn.io/docs/
- **Go Standard Library**: https://pkg.go.dev/std
- **Project README and issues**: Check GitHub for known patterns

### 4. Use Incremental Development

Break down AI-assisted development into small, verifiable steps:

1. **Start small**: Ask for one function or one fix at a time
2. **Build and test immediately**: Run `make ci` after each change
3. **Validate behavior**: Check that the code works as intended
4. **Review diffs carefully**: Use `git diff` to ensure only intended changes
5. **Commit frequently**: Small commits make it easier to identify issues

### 5. Be Specific About Project Architecture

The Longhorn SPDK Engine has specific architectural patterns:

- **Client-Server Architecture**: pkg/client and pkg/spdk/server.go
- **Disk Management**: pkg/spdk/disk/ with multiple driver implementations
- **Type Safety**: Strong typing in pkg/types/ and pkg/api/
- **Error Handling**: Consistent error wrapping patterns

When asking AI for help, reference these patterns explicitly:

```
✅ "Following the error handling pattern used in pkg/spdk/engine.go lines 
1653-1655, how should I handle RAID recreation failures in my new function?"
```

### 6. Question Confident but Unusual Suggestions

Be skeptical if an AI suggests:

- **Unusual imports** not present in go.mod
- **Deprecated patterns** (check git history)
- **Non-idiomatic Go** (run `gofmt` and `golangci-lint`)
- **Breaking changes** to public APIs without justification
- **Complex solutions** when simpler ones exist in the codebase

### 7. Validate Generated Comments and Documentation

AI-generated comments can be misleading:

- **Verify technical accuracy**: Does it match what the code actually does?
- **Check for outdated information**: Does it reference old API versions?
- **Ensure clarity**: Would another developer understand it?
- **Match project style**: Look at existing comments for tone and format

### 8. Test Edge Cases and Error Paths

AI often focuses on happy paths. Always add:

- **Nil checks** for pointers
- **Error handling** for all failure scenarios
- **Boundary conditions** (empty lists, zero values, etc.)
- **Concurrent access** patterns (this project uses locks extensively)

### 9. Use AI for Understanding, Not Just Generating

Leverage AI to:

- **Explain existing code** patterns in the repository
- **Understand error messages** in context
- **Learn about SPDK concepts** you're unfamiliar with
- **Review your own code** before submitting

Then **verify the explanations** by reading the actual code and documentation.

### 10. Maintain Human Oversight

**Never blindly accept AI-generated code.** Always:

- ✅ Read and understand every line
- ✅ Run the full test suite
- ✅ Check for security vulnerabilities
- ✅ Review for performance implications (especially in I/O paths)
- ✅ Ensure thread-safety (many operations use mutexes)

## Specific Patterns in This Project

### RAID Management

When working with RAID functionality, be aware:

- RAID bdevs are not persistent (see pkg/spdk/server.go:236)
- RAID operations require specific timeout configurations (see pkg/spdk/types.go:46-54)
- RAID recreation is a critical path (see pkg/spdk/engine.go:1358+)

If an AI suggests RAID-related code, verify against these existing implementations.

### Lock Patterns

The codebase uses extensive locking:

```go
// Common pattern from recent fixes
func (r *Replica) someOperation() error {
    r.Lock()
    defer r.Unlock()
    
    // ... operation
}
```

AI suggestions must respect existing lock hierarchies to avoid deadlocks.

### Error Handling

Consistent error wrapping pattern:

```go
if err != nil {
    return errors.Wrapf(err, "failed to operation on %s", name)
}
```

Don't let AI introduce inconsistent error handling styles.

## Verification Checklist

Before committing AI-assisted code:

- [ ] Code compiles without errors (`make build`)
- [ ] All tests pass (`make test`)
- [ ] Linters are satisfied (`make lint`)
- [ ] No new security vulnerabilities introduced
- [ ] Changes match existing code style and patterns
- [ ] Comments accurately describe functionality
- [ ] Error paths are properly handled
- [ ] Thread-safety is maintained
- [ ] No breaking changes to public APIs
- [ ] Documentation is updated if needed

## When to Seek Human Review

Always request human code review when:

- Making changes to critical paths (I/O operations, RAID management)
- Modifying public APIs or type definitions
- Introducing new dependencies
- Implementing security-sensitive features
- Unsure about AI suggestions after verification attempts
- Changes affect multiple subsystems

## Resources

- [Longhorn SPDK Engine Repository](https://github.com/longhorn/longhorn-spdk-engine)
- [SPDK Official Documentation](https://spdk.io/doc/)
- [Effective Go](https://go.dev/doc/effective_go)
- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)

## Contributing

If you discover AI-generated code that doesn't follow these guidelines or introduces issues, please:

1. Document the problem in a GitHub issue
2. Reference the specific AI suggestion that caused the problem
3. Propose improvements to these guidelines

By following these practices, we can leverage AI assistance effectively while maintaining the high quality and reliability expected in storage infrastructure software.

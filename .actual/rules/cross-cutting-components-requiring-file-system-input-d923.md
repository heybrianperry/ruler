# Adoption of fs/promises for Asynchronous File System Operations: Components Requiring File System Input Output

These rules are ALWAYS ACTIVE for all internal modules, command-line handlers, agent orchestrators, and protocol propagation components performing file system access.

### Rules

- **R-FS-001** MUST: Components requiring file system input and output MUST utilize the fs/promises module for all non-blocking asynchronous file operations.

### Verify

```bash
# Discover project test runner and execute automated test suite
# Discover static analysis tool and run linting checks to identify disallowed synchronous file system calls
```

**Accept when:**
- All automated test suites pass without regression in asynchronous execution.
- Static analysis validates that no synchronous file system calls exist in asynchronous execution paths.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
# Adoption of fs/promises for Asynchronous File System Operations: File Paths Supplied Promises Calls Resolved

These rules are ALWAYS ACTIVE for all internal modules, command-line handlers, agent orchestrators, and protocol propagation components performing file system access.

### Rules

- **R-FS-001** SHOULD: File paths supplied to fs/promises calls SHOULD be resolved through standard path resolution utilities prior to invoking file system actions.

### Verify

```bash
# Discover the project test runner from repository configuration and execute the automated test suite.
# Discover the static analysis tool from repository configuration and run linting checks to identify disallowed synchronous file system calls.
```

**Accept when:**
- All automated test suites pass without regression in asynchronous execution.
- Static analysis validates that no synchronous file system calls exist in asynchronous execution paths.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
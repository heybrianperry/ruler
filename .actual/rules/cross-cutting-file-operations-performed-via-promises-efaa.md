# Adoption of fs/promises for Asynchronous File System Operations: File Operations Performed Via Promises Use

These rules are ALWAYS ACTIVE for all internal modules, command-line handlers, agent orchestrators, protocol propagation components, and any operations reading, writing, updating, or querying the host file system.

### Rules

- **R-FS-001** MUST: File operations performed via fs/promises MUST use async and await syntax or standard promise resolution chains rather than synchronous execution alternatives.

### Verify

```bash
# Discover the project test runner from repository configuration and execute the automated test suite.
# Discover the static analysis tool from repository configuration and run linting checks to identify disallowed synchronous file system calls.
```

**Accept when:**
- All automated test suites pass without regression in asynchronous execution.
- Static analysis validates that no synchronous file system calls exist in asynchronous execution paths.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory via automated static analysis checks in the continuous integration pipeline and peer code review.
</enforcement>
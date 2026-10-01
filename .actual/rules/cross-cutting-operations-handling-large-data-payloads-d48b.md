# Adoption of fs/promises for Asynchronous File System Operations: Operations Handling Large Data Payloads Evaluate

These rules are ALWAYS ACTIVE for all internal modules, command-line handlers, agent orchestrators, and protocol propagation components performing file system access or reading, writing, updating, or querying the host file system.

### Rules

- **R-FS-001** SHOULD: Operations handling large data payloads SHOULD evaluate stream-oriented interfaces rather than loading complete file contents into memory with single promise invocations.

### Verify

```bash
# Discover the project test runner from repository configuration and execute the automated test suite.
# Discover the static analysis tool from repository configuration and run linting checks to identify disallowed synchronous file system calls.
```

**Accept when:**
- All automated test suites pass without regression in asynchronous execution.
- Static analysis validates that no synchronous file system calls exist in asynchronous execution paths.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory via automated static analysis checks and peer code review.
</enforcement>
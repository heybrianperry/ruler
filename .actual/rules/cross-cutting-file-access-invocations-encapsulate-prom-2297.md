# Adoption of fs/promises for Asynchronous File System Operations: File Access Invocations Encapsulate Promise Rejections

These rules are ALWAYS ACTIVE for all agent implementation modules and core processor/utility modules performing file reading, parsing, or writing.

### Rules

- **R-FS-001** SHOULD: File access invocations SHOULD encapsulate promise rejections with explicit domain error handling when reading configuration or specification files.

### Verify

```bash
# Discover the repository test runner script from the project manifest and execute the test suite covering file I/O operations.
# Run static analysis checks defined in the project build configuration to detect unapproved synchronous file system calls.
```

**Accept when:**
- All file operations in agent and core processing modules utilize fs/promises.
- Static analysis and automated test suites pass without detecting blocking file operations.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
# Adoption of fs/promises for Asynchronous File System Operations: Components Not Execute Blocking Synchronous File

These rules are ALWAYS ACTIVE for all agent implementation modules, core processors, and utility modules performing file reading, parsing, or writing.

### Rules

- **R-FS-001** MUST_NOT: Components MUST_NOT execute blocking synchronous file system operations within agent workflows or core processor pipelines.

### Verify

```bash
# Discover the repository test runner script from the project manifest and execute the test suite covering file I/O operations.
# Run static analysis checks defined in the project build configuration to detect unapproved synchronous file system calls.
```

**Accept when:**
- All file operations in agent and core processing modules utilize fs/promises.
- Static analysis and automated test suites pass without detecting blocking file operations.

<enforcement>
Claude Code MUST NOT skip or defer verification. All file access must be verified through automated static analysis and test suites.
</enforcement>
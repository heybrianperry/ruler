# Adoption of fs/promises for Asynchronous File System Operations: Components Deserializing File Content Retrieved Via

These rules are ALWAYS ACTIVE for all agent implementation modules, core processors, and utility modules requiring disk-based asset loading, configuration access, or file reading/writing/parsing.

### Rules

- **R-FSP-001** SHOULD: Components deserializing file content retrieved via fs/promises SHOULD validate the raw data structure before passing parsed payloads to downstream processors.

### Verify

```bash
# Discover the repository test runner script from the project manifest and execute the test suite covering file I/O operations.
# Run static analysis checks defined in the project build configuration to detect unapproved synchronous file system calls.
```

**Accept when:**
- All file operations in agent and core processing modules utilize fs/promises.
- Static analysis and automated test suites pass without detecting blocking file operations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated static analysis checks and mandatory peer review.
</enforcement>
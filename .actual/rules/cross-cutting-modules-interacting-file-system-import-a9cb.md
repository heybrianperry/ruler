# Adoption of fs/promises for Asynchronous File System Operations: Modules Interacting File System Import Use

These rules are ALWAYS ACTIVE for all modules interacting with the file system across agent implementation modules, core processors, and utility modules performing file reading, parsing, or writing.

### Rules

- **R-FS-001** MUST: Modules interacting with the file system MUST import and use `fs/promises` for all file read, write, and inspection operations to ensure non-blocking asynchronous execution.

### Verify

```bash
# Discover and run the project's test suite covering file I/O operations
# Discover and run static analysis checks to detect unapproved synchronous file system calls
```

**Accept when:**
- All file operations in agent and core processing modules utilize `fs/promises`.
- Static analysis and automated test suites pass without detecting blocking file operations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated static analysis checks in CI pipelines and mandatory peer reviews.
</enforcement>
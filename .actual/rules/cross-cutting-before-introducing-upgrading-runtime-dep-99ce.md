# Adoption of fs/promises for Asynchronous File System Operations: Before Introducing Upgrading Runtime Dependencies Utilizing

These rules are ALWAYS ACTIVE for all internal modules, command-line handlers, agent orchestrators, and protocol propagation components performing file system access or interacting with host storage.

### Rules

- **R-FS-001** MUST: Before introducing or upgrading runtime dependencies or utilizing module APIs, developers MUST discover the authoritative repository lock artifact and verify the exact resolved runtime environment version.
- **R-FS-002** MUST: Use asynchronous `fs/promises` interfaces for all host file system operations to prevent blocking execution on the primary thread.
- **R-FS-003** MUST: Mandate structured try-catch or promise rejection handlers across all file access boundaries to catch missing files, permission errors, and invalid path inputs.
- **R-FS-004** MUST: Enforce streaming or chunked processing strategies for arbitrarily large data files to prevent memory exhaustion.

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
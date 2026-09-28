# Adoption of fs/promises for Asynchronous File System Operations: Developers Inspect Project Resolution Artifact Confirm

These rules are ALWAYS ACTIVE for all agent implementation modules, core processor modules, and utility modules requiring disk-based asset loading, configuration access, or file reading/parsing/writing.

### Rules

- **R-FSP-001** MUST: Developers MUST inspect the project resolution artifact to confirm the active runtime and dependency constraints before introducing or modifying file system integration code.
- **R-FSP-002** MUST: All file operations in agent and core processing modules utilize fs/promises.
- **R-FSP-003** MUST: Encapsulate fs/promises calls within dedicated utility modules or domain processors to centralize path resolution and encoding defaults.
- **R-FSP-004** MUST: Pair file content retrieval with schema validation to verify file contents conform to expected formats before processing.
- **R-FSP-005** MUST: Encapsulate file system promise calls in structured error-handling blocks with domain-specific recovery mechanisms to prevent unhandled promise rejections.
- **R-FSP-006** MUST: Coordinate shared file access through sequential processing stages or designated utility wrappers to avoid data corruption from concurrent read and write operations.

### Verify

```bash
# Discover and run the repository test runner script covering file I/O operations
# (Derived from project manifest)

# Run static analysis checks defined in the project build configuration to detect unapproved synchronous file system calls
```

**Accept when:**
- All file operations in agent and core processing modules utilize fs/promises.
- Static analysis and automated test suites pass without detecting blocking file operations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated static analysis checks in CI pipelines and mandatory peer review.
</enforcement>
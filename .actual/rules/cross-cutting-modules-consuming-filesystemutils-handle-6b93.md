# FileSystemUtils Internal Module for Centralized Filesystem Operations: Modules Consuming Filesystemutils Handle Operational Errors

These rules are ALWAYS ACTIVE for components implementing agent adapters, path helpers, and configuration propagation handlers that require filesystem interaction, as well as modules performing configuration file inspection, structured document parsing, and directory resolution.

### Rules

- **R-FSU-001** MUST: Modules consuming FileSystemUtils MUST handle operational errors, including missing targets and parsing failures, cleanly without terminating the surrounding execution context.

### Verify

```bash
# Discover and execute project verification test suites covering filesystem utility functions and consuming agent adapters.
# Run repository static analysis and linting checks to confirm module imports adhere to internal architectural boundaries.
```

**Accept when:**
- All unit and integration tests exercising FileSystemUtils and dependent agent adapters complete with zero errors.
- Static boundary analysis confirms that all filesystem interactions within agent and propagation modules route through approved internal utilities.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
# FileSystemUtils Internal Module for Centralized Filesystem Operations: Agent Definitions Configuration Propagators Not Bypass

These rules are ALWAYS ACTIVE for components implementing agent adapters, path helpers, and configuration propagation handlers that require filesystem interaction, as well as modules performing configuration file inspection, structured document parsing, and directory resolution.

### Rules

- **R-FS-001** MUST_NOT: Agent definitions and configuration propagators MUST NOT bypass FileSystemUtils to execute redundant custom path resolution or direct filesystem mutations that the utility module already standardizes.

### Verify

```bash
# Discover and execute project verification test suites covering filesystem utility functions and consuming agent adapters.
# Run repository static analysis and linting checks to confirm module imports adhere to internal architectural boundaries.
```

**Accept when:**
- All unit and integration tests exercising FileSystemUtils and dependent agent adapters complete with zero errors.
- Static boundary analysis confirms that all filesystem interactions within agent and propagation modules route through approved internal utilities.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated continuous integration test, static analysis pipelines, and peer code review.
</enforcement>
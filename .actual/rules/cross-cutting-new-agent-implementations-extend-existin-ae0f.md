# FileSystemUtils Internal Module for Centralized Filesystem Operations: New Agent Implementations Extend Existing Abstraction

These rules are ALWAYS ACTIVE for components implementing agent adapters, path helpers, and configuration propagation handlers that require filesystem interaction, as well as modules performing configuration file inspection, structured document parsing, and directory resolution.

### Rules

- **R-FSU-001** SHOULD: New agent implementations SHOULD extend existing abstraction boundaries by delegating common path resolution and disk reading workflows to FileSystemUtils.

### Verify

```bash
# Discover and execute project verification test suites covering filesystem utility functions and consuming agent adapters.
# Run repository static analysis and linting checks to confirm module imports adhere to internal architectural boundaries.
```

**Accept when:**
- All unit and integration tests exercising FileSystemUtils and dependent agent adapters complete with zero errors.
- Static boundary analysis confirms that all filesystem interactions within agent and propagation modules route through approved internal utilities.

<enforcement>
Claude Code MUST NOT skip or defer verification. All changes involving filesystem access or agent implementations must be verified through automated continuous integration tests and peer code reviews.
</enforcement>
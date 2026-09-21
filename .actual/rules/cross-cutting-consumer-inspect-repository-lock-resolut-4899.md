# FileSystemUtils Internal Module for Centralized Filesystem Operations: Consumer Inspect Repository Lock Resolution Artifacts

These rules are ALWAYS ACTIVE for components implementing agent adapters, path helpers, configuration propagation handlers, and modules performing configuration file inspection, structured document parsing, and directory resolution that require filesystem interaction.

### Rules

- **R-FS-001** MUST: The consumer MUST inspect repository lock and resolution artifacts to confirm the exact resolved versions of all runtime filesystem and serialization dependencies before introducing or altering filesystem utility calls.

### Verify

```bash
# Discover and execute project verification test suites covering filesystem utility functions and consuming agent adapters
# Run repository static analysis and linting checks to confirm module imports adhere to internal architectural boundaries
```

**Accept when:**
- All unit and integration tests exercising FileSystemUtils and dependent agent adapters complete with zero errors.
- Static boundary analysis confirms that all filesystem interactions within agent and propagation modules route through approved internal utilities.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
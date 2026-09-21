# FileSystemUtils Internal Module for Centralized Filesystem Operations: Agent Adapters Path Resolvers Protocol Propagation

These rules are ALWAYS ACTIVE for agent adapters, path resolvers, protocol propagation modules, and configuration handlers requiring filesystem interaction.

### Rules

- **R-FSU-001** MUST: All agent adapters, path resolvers, and protocol propagation modules MUST route shared filesystem interactions and path normalization through the internal FileSystemUtils module rather than reimplementing ad-hoc disk operations.

### Verify

```bash
# Discover and execute project verification test suites covering filesystem utility functions and consuming agent adapters
# Run repository static analysis and linting checks to confirm module imports adhere to internal architectural boundaries
```

**Accept when:**
- All unit and integration tests exercising FileSystemUtils and dependent agent adapters complete with zero errors.
- Static boundary analysis confirms that all filesystem interactions within agent and propagation modules route through approved internal utilities.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests introducing direct unapproved filesystem calls or bypassing the utility module will be blocked until refactored.
</enforcement>
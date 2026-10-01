# FileSystemUtils Core Module Adoption for Centralized Filesystem Operations: Consumer Discover Project Dependency Manifest Lock

These rules are ALWAYS ACTIVE for all internal services, command handlers, agent adapters, and configuration loaders performing filesystem access, as well as directory resolution routines inspecting operating system environment variables for configuration paths.

### Rules

- **R-FSU-001** MUST: The consumer MUST discover the project dependency manifest and lock artifact to resolve all dependency versions prior to implementing or extending module integrations.

### Verify

```bash
# Discover the project static analysis and linting scripts from project configuration and execute them to verify import boundaries.
# Discover the project test runner through repository configuration and execute the full test suite covering filesystem utility operations.
```

**Accept when:**
- All unit and integration tests for the centralized filesystem utility and consuming modules pass without errors.
- Static analysis confirms zero unauthorized direct runtime filesystem imports across domain packages.
- Verification confirms that components across agent, settings, path resolution, and configuration domains route operations through the centralized module.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated continuous integration workflows and peer code reviews enforce these rules, and violations will cause build failures.
</enforcement>
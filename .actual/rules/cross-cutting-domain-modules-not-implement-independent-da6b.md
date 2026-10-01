# FileSystemUtils Core Module Adoption for Centralized Filesystem Operations: Domain Modules Not Implement Independent Environment

These rules are ALWAYS ACTIVE for all internal services, command handlers, agent adapters, configuration loaders, and directory resolution routines inspecting operating system environment variables for configuration paths.

### Rules

- **R-FSU-001** MUST_NOT: Domain modules MUST NOT implement independent environment variable inspection routines for configuration directory resolution when equivalent capabilities exist within FileSystemUtils.

### Verify

```bash
# Discover and execute project static analysis and linting scripts to verify import boundaries
# Discover and execute project test runner covering filesystem utility operations
```

**Accept when:**
- All unit and integration tests for the centralized filesystem utility and consuming modules pass without errors.
- Static analysis confirms zero unauthorized direct runtime filesystem imports across domain packages.
- Verification confirms that components across agent, settings, path resolution, and configuration domains route operations through the centralized module.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory.
</enforcement>
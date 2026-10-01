# FileSystemUtils Core Module Adoption for Centralized Filesystem Operations: Application Modules Requiring Filesystem Inspection Directory

These rules are ALWAYS ACTIVE for all application modules, internal services, command handlers, agent adapters, and configuration loaders requiring filesystem inspection, directory traversal, or path resolution.

### Rules

- **R-FS-001** MUST: All application modules requiring filesystem inspection, directory traversal, or path resolution MUST route operations through the centralized FileSystemUtils internal module rather than directly invoking low-level runtime filesystem bindings.

### Verify

```bash
# Discover and execute project static analysis and linting scripts to verify import boundaries
# Discover and execute the full test suite covering filesystem utility operations
```

**Accept when:**
- All unit and integration tests for the centralized filesystem utility and consuming modules pass without errors.
- Static analysis confirms zero unauthorized direct runtime filesystem imports across domain packages.
- Verification confirms that components across agent, settings, path resolution, and configuration domains route operations through the centralized module.

<enforcement>
Claude Code MUST NOT skip or defer verification. Continuous integration checks and peer reviews enforce that all filesystem interactions route through the centralized utility module.
</enforcement>
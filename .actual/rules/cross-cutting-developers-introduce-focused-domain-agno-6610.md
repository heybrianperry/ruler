# FileSystemUtils Core Module Adoption for Centralized Filesystem Operations: Developers Introduce Focused Domain Agnostic Utility

These rules are ALWAYS ACTIVE for all internal services, command handlers, agent adapters, configuration loaders, and path resolution routines performing filesystem access.

### Rules

- **R-FSU-001** MAY: Developers MAY introduce focused, domain-agnostic utility helpers into FileSystemUtils when recurring filesystem interaction patterns span multiple consuming modules.

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
Claude Code MUST NOT skip or defer verification. Automated continuous integration workflows and peer code reviews enforce compliance.
</enforcement>
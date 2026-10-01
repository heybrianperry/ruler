# FileSystemUtils Core Module Adoption for Centralized Filesystem Operations: Filesystemutils Provide Standardized Error Logging Diagnostic

These rules are ALWAYS ACTIVE for all internal services, command handlers, agent adapters, and configuration loaders performing filesystem access, as well as directory resolution routines inspecting operating system environment variables for configuration paths.

### Rules

- **R-FSU-001** SHOULD: FileSystemUtils SHOULD provide standardized error logging and diagnostic messaging for failed filesystem access and missing directory states.
- **R-FSU-002** MANDATORY: Consuming modules MUST import functional capabilities directly from the centralized utility rather than duplicating path manipulation and inspection logic.
- **R-FSU-003** MANDATORY: All environment variable lookups for user configuration directories MUST follow the standardized fallback chain implemented within the core utility.
- **R-FSU-004** MANDATORY: Prior to writing code that uses a versioned library, developers MUST find the dependency manifest, identify the build tool, inspect the repository lock or resolution artifact to determine the exact resolved version, look up official documentation for that exact version, confirm every API/class/function exists in that version's documentation, and re-run version grounding steps per dependency at point of use for version-sensitive behavior.

### Verify

```bash
# Discover and execute project static analysis and linting scripts from project configuration to verify import boundaries
# Discover the project test runner through repository configuration and execute the full test suite covering filesystem utility operations
```

**Accept when:**
- All unit and integration tests for the centralized filesystem utility and consuming modules pass without errors.
- Static analysis confirms zero unauthorized direct runtime filesystem imports across domain packages.
- Verification confirms that components across agent, settings, path resolution, and configuration domains route operations through the centralized module.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory.
</enforcement>
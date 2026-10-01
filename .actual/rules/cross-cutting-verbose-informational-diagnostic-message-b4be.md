# Console API Standard Stream Logging: Verbose Informational Diagnostic Messages Include Module

These rules are ALWAYS ACTIVE for all command-line interfaces, core modules, and local file system or configuration workflows.

### Rules

- **R-LOG-001** SHOULD: Verbose and informational diagnostic messages SHOULD include module identification tags to distinguish subsystem origins.
- **R-LOG-002** MANDATORY: Rely on built-in console error and warning APIs to eliminate external dependency overhead and maintain readable diagnostic tracing.
- **R-LOG-003** MANDATORY: Route all failure details through designated error formatting helpers prior to emission.
- **R-LOG-004** MANDATORY: Ensure module prefixes are applied consistently across all console warning and error invocations.

### Verify

```bash
# Discover repository test and lint scripts from the project manifest and execute the static analysis suite.
# Execute the project verification script discovered from the build configuration to inspect logging stream compliance.
```

**Accept when:**
- Accept when all operational errors and warnings are emitted exclusively via console error and warning APIs with standardized prefixes.
- Accept when project verification and linting commands execute without logging-related violations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by static analysis, lint checks configured in CI, and peer review of pull requests touching diagnostic logging and error handling.
</enforcement>
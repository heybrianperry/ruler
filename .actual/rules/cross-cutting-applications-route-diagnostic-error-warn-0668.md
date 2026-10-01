# Console API Standard Stream Logging: Applications Route Diagnostic Error Warning Messages

These rules are ALWAYS ACTIVE for command-line interfaces, core utility modules, and local file system or configuration workflows requiring operational diagnostic reporting.

### Rules

- **R-LOG-001** MUST: Applications MUST route diagnostic error and warning messages through the runtime console error and warning APIs using standardized prefix formatting.
- **R-LOG-002** MUST: Route all failure details through designated error formatting helpers prior to emission.
- **R-LOG-003** MUST: Ensure module prefixes are applied consistently across all console warning and error invocations.

### Verify

```bash
# Discover repository test and lint scripts from the project manifest and execute the static analysis suite.
# Execute the project verification script discovered from the build configuration to inspect logging stream compliance.
```

**Accept when:**
- All operational errors and warnings are emitted exclusively via console error and warning APIs with standardized prefixes.
- Project verification and linting commands execute without logging-related violations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests containing unformatted or non-standard stream output must be updated before merging.
</enforcement>
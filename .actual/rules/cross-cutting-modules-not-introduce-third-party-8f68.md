# Console API Standard Stream Logging: Modules Not Introduce Third Party Logging

These rules are ALWAYS ACTIVE for diagnostic, warning, and error reporting across command-line interfaces and core modules.

### Rules

- **R-LOG-001** MUST_NOT: Modules MUST NOT introduce third-party logging dependencies when built-in runtime console logging satisfies diagnostic reporting requirements.

### Verify

```bash
# Discover repository test and lint scripts from the project manifest and execute the static analysis suite.
# Execute the project verification script discovered from the build configuration to inspect logging stream compliance.
```

**Accept when:**
- Accept when all operational errors and warnings are emitted exclusively via console error and warning APIs with standardized prefixes.
- Accept when project verification and linting commands execute without logging-related violations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by static analysis, lint checks, and peer review. Pull requests containing unformatted or non-standard stream output must be updated before merging.
</enforcement>
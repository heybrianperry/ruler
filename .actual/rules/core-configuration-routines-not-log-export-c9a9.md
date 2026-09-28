# process.env for Configuration Directory Resolution: Configuration Routines Not Log Export Expose

These rules are ALWAYS ACTIVE for all configuration path resolution and environment variable lookup routines within the configuration subsystem.

### Rules

- **R-CFG-001** MUST_NOT: Configuration routines MUST NOT log, export, or expose environment variables accessed during directory resolution to external monitoring sinks.

### Verify

```bash
# Discover the repository test runner from the project manifest and execute the configuration loading test suite.
# Inspect the project scripts to discover and run the static analysis security scanner for environment variable handling.
```

**Accept when:**
- Configuration resolution tests pass across all supported environments using default and custom path overrides.
- Security analysis reports zero credential leakage or unauthorized environment exposure findings.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated test suites in continuous integration and peer code review during pull requests.
</enforcement>
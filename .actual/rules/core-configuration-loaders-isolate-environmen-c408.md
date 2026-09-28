# process.env for Configuration Directory Resolution: Configuration Loaders Isolate Environment Variable Lookups

These rules are ALWAYS ACTIVE for configuration path resolution and environment variable lookup within the configuration subsystem.

### Rules

- **R-CFG-001** SHOULD: Configuration loaders isolate environment variable lookups behind a centralized configuration interface rather than reading global runtime environment properties directly across disparate modules.

### Verify

```bash
# Discover the repository test runner from the project manifest and execute the configuration loading test suite.
# Inspect the project scripts to discover and run the static analysis security scanner for environment variable handling.
```

**Accept when:**
- Configuration resolution tests pass across all supported environments using default and custom path overrides.
- Security analysis reports zero credential leakage or unauthorized environment exposure findings.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
# process.env for Configuration Directory Resolution: Configuration Loading Component Access Process Env

These rules are ALWAYS ACTIVE for configuration path resolution and environment variable lookup within the configuration subsystem.

### Rules

- **R-CFG-001** MUST: The configuration loading component MUST access process.env for directory resolution only when locating user-specific configuration paths and MUST NOT treat general environment paths as managed credentials.

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
# Use of process.env.XDG_CONFIG_HOME for Secret Handling: Configuration Loaded External Sources Undergo Explicit

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-XDG-001** MUST: All configuration loaded from external sources MUST undergo explicit input validation using parsing functions such as parseTOML, JSON.parse, or similar mechanisms.

### Verify

```bash
# Discover the project's build scripts and execute the configuration loading tests.
# Inspect the project's environment setup documentation for XDG_CONFIG_HOME configuration.
# Run the application with intentionally malformed configuration files to verify error handling.
```

**Accept when:**
- All configuration loading tests pass successfully.
- The application correctly loads sensitive configuration from XDG_CONFIG_HOME at runtime.
- Invalid configuration inputs are gracefully handled, and appropriate errors are reported.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
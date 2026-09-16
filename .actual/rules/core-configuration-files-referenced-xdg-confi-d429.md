# Use of process.env.XDG_CONFIG_HOME for Secret Handling: Configuration Files Referenced Xdg Config Home

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-XDG-CONFIG-001** SHOULD: Configuration files referenced by XDG_CONFIG_HOME SHOULD be protected with appropriate file system permissions to restrict unauthorized access.

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
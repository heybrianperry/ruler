# Use of process.env.XDG_CONFIG_HOME for Secret Handling: Sensitive Configuration Secrets Retrieved Environment Variables

These rules are ALWAYS ACTIVE for all modules responsible for application configuration loading and components handling sensitive data or credentials.

### Rules

- **R-SCS-001** MUST: Sensitive configuration and secrets MUST be retrieved from environment variables, specifically utilizing process.env.XDG_CONFIG_HOME for user-specific configuration paths.

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
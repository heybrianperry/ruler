# Use of process.env.XDG_CONFIG_HOME for Secret Handling: Before Implementing Modifying Code That Interacts

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- MUST: Before implementing or modifying code that interacts with Node.js environment variables, the consumer MUST discover the project's dependency manifest and lock file to determine the exact resolved Node.js version and consult its official documentation for process.env behavior.

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
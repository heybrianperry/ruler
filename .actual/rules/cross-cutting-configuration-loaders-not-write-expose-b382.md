# XDG_CONFIG_HOME Environment Path Resolution for Configuration and Credential Storage: Configuration Loaders Not Write Expose Sensitive

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-XDG-001** MUST_NOT: Configuration loaders MUST NOT write or expose sensitive credentials when logging path resolution errors or configuration diagnostic events.

### Verify

```bash
# Discover and execute the project unit test suite for configuration and path resolution utilities.
# Discover and execute the repository static analysis and linting scripts to verify adherence to centralized path utilities.
```

**Accept when:**
- Configuration path resolution tests pass, demonstrating correct precedence of process.env.XDG_CONFIG_HOME over default user directory fallbacks.
- Static analysis confirms no unencapsulated reads of the environment variable exist outside core filesystem utility modules.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
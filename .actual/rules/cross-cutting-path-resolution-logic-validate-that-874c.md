# XDG_CONFIG_HOME Environment Path Resolution for Configuration and Credential Storage: Path Resolution Logic Validate That Resolved

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-PATH-001** MUST: Path resolution logic MUST validate that resolved base directories exist and maintain appropriate filesystem permissions before reading or persisting credential files.

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
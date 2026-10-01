# XDG_CONFIG_HOME Environment Path Resolution for Configuration and Credential Storage: Modules That Rely Third Party Runtime

These rules are ALWAYS ACTIVE for modules that rely on third-party runtime dependencies, global configuration files, user preferences, and credential storage directories.

### Rules

- **R-XDG-001** MUST: Modules that rely on third-party runtime dependencies MUST discover the ecosystem lock file and verify the exact resolved version prior to execution.

### Verify

```bash
# Discover and execute the project unit test suite for configuration and path resolution utilities
# Discover and execute the repository static analysis and linting scripts to verify adherence to centralized path utilities
```

**Accept when:**
- Configuration path resolution tests pass, demonstrating correct precedence of process.env.XDG_CONFIG_HOME over default user directory fallbacks.
- Static analysis confirms no unencapsulated reads of the environment variable exist outside core filesystem utility modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated continuous integration pipeline testing configuration resolution suites and peer code review.
</enforcement>
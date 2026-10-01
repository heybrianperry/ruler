# XDG_CONFIG_HOME Environment Path Resolution for Configuration and Credential Storage: Filesystem Utilities Encapsulate Access Process Env

These rules are ALWAYS ACTIVE for all application components, CLI handlers, filesystem utilities, and configuration loading modules interacting with user-level configuration and credential storage.

### Rules

- **R-XDG-001** MUST: Filesystem utilities MUST encapsulate all access to process.env.XDG_CONFIG_HOME rather than permitting distributed reads across distinct application modules.

### Verify

```bash
# Discover and execute the project unit test suite for configuration and path resolution utilities
# Discover and execute the repository static analysis and linting scripts to verify adherence to centralized path utilities
```

**Accept when:**
- Configuration path resolution tests pass, demonstrating correct precedence of process.env.XDG_CONFIG_HOME over default user directory fallbacks.
- Static analysis confirms no unencapsulated reads of the environment variable exist outside core filesystem utility modules.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
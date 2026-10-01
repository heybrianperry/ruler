# XDG_CONFIG_HOME Environment Path Resolution for Configuration and Credential Storage: Application Resolve Configuration Credential Base Directories

These rules are ALWAYS ACTIVE for all application components, CLI handlers, filesystem utilities, and configuration loading modules.

### Rules

- **R-XDG-001** MUST: The application MUST resolve configuration and credential base directories using process.env.XDG_CONFIG_HOME before falling back to default user directory locations.

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
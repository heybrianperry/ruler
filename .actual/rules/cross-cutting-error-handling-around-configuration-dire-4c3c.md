# Adopt XDG Base Directory Specification for Configuration File Location: Error Handling Around Configuration Directory Access

These rules are ALWAYS ACTIVE for all configuration file loading operations at application startup, CLI handlers that initialize configuration subsystems, file system utilities that resolve user or system configuration paths, and configuration parsers that need to locate configuration files before reading them.

### Rules

- **R-XDG-001** SHOULD: Error handling around configuration directory access SHOULD log the resolved path to aid debugging when XDG environment variables are misconfigured.

### Verify

```bash
# Discover the project test suite and execute tests covering configuration loading with XDG_CONFIG_HOME set to a custom path
# Discover the project test suite and execute tests covering configuration loading with XDG_CONFIG_HOME unset to verify fallback behavior
# Inspect the codebase for all modules accessing environment variables for configuration paths and verify they use the centralized XDG-aware resolution mechanism
```

**Accept when:**
- All configuration loading tests pass with both XDG_CONFIG_HOME set and unset
- Configuration path resolution is consistent across CLI handlers, file system utilities, and configuration loading modules
- Error messages clearly indicate the resolved configuration directory path when configuration loading fails

<enforcement>
Clause Code MUST NOT skip or defer verification. Configuration path resolution must be centralized in a shared utility module, fallback logic must follow XDG Base Directory specification defaults, and resolved paths must be logged during initialization.
</enforcement>
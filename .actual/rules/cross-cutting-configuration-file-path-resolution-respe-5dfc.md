# Adopt XDG Base Directory Specification for Configuration File Location: Configuration File Path Resolution Respect Xdg

These rules are ALWAYS ACTIVE for all configuration file loading operations at application startup, CLI handlers that initialize configuration subsystems, file system utilities that resolve user or system configuration paths, and configuration parsers that need to locate configuration files before reading them.

### Rules

- **R-XDG-001** MUST: Configuration file path resolution MUST respect the XDG_CONFIG_HOME environment variable when present, implementing the XDG Base Directory specification for configuration file location.
- **R-XDG-002** MUST: Centralize XDG path resolution logic in a shared utility module to ensure consistent behavior across CLI handlers, file system utilities, and configuration loaders.
- **R-XDG-003** MUST: Implement fallback logic that follows XDG Base Directory specification defaults when XDG_CONFIG_HOME is unset, typically defaulting to a subdirectory within the user home directory.
- **R-XDG-004** MUST: Log the resolved configuration directory path during initialization to aid debugging when users have custom XDG environment variable configurations.
- **R-XDG-005** MUST: Verify all configuration path resolution uses the centralized XDG-aware utility through code review.
- **R-XDG-006** MUST: Implement integration tests covering configuration loading with various XDG environment variable configurations.
- **R-XDG-007** MUST: Use static analysis to detect direct hardcoded configuration paths that bypass XDG resolution.

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
- Code review verifies all configuration path resolution uses the centralized XDG-aware utility
- Integration tests detect no inconsistent configuration path resolution
- Static analysis finds no direct hardcoded configuration paths that bypass XDG resolution

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration file path resolution MUST use the centralized XDG-aware utility. Violations require code review feedback and refactoring. CI pipeline MUST fail if integration tests detect inconsistent configuration path resolution.
</enforcement>
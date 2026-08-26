# Adopt XDG Base Directory Specification for Configuration File Location: Configuration Path Resolution Provide Fallback Behavior

These rules are ALWAYS ACTIVE for all configuration file loading operations at application startup, CLI handlers that initialize configuration subsystems, file system utilities that resolve user or system configuration paths, and configuration parsers that need to locate configuration files before reading them.

### Rules

- **R-XDG-001** MUST: Configuration path resolution MUST provide fallback behavior when XDG_CONFIG_HOME is not set, following XDG Base Directory specification defaults.
- **R-XDG-002** MUST: Centralize XDG path resolution logic in a shared utility module to ensure consistent behavior across CLI handlers, file system utilities, and configuration loaders.
- **R-XDG-003** MUST: Implement fallback logic that follows XDG Base Directory specification defaults when XDG_CONFIG_HOME is unset, typically defaulting to a subdirectory within the user home directory.
- **R-XDG-004** MUST: Log the resolved configuration directory path during initialization to aid debugging when users have custom XDG environment variable configurations.
- **R-XDG-005** MUST: Verify all configuration path resolution uses the centralized XDG-aware utility through code review.
- **R-XDG-006** MUST: Ensure error messages clearly indicate the resolved configuration directory path when configuration loading fails.
- **R-XDG-007** SHOULD: Document platform-specific behavior for Windows and macOS where XDG Base Directory specification is not natively supported.
- **R-XDG-008** SHOULD: Implement fallback to platform-appropriate defaults when XDG variables are unset on non-Unix systems.

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
- Static analysis detects no direct hardcoded configuration paths that bypass XDG resolution
- All configuration loading paths use the centralized XDG path resolution utility

<enforcement>
Code review MUST verify all configuration path resolution uses the centralized XDG-aware utility. Integration tests MUST cover configuration loading with various XDG environment variable configurations. Static analysis MUST detect direct hardcoded configuration paths that bypass XDG resolution. CI pipeline MUST fail if integration tests detect inconsistent configuration path resolution. Claude Code MUST NOT skip or defer verification.
</enforcement>
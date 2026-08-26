# Adopt XDG Base Directory Specification for Configuration File Location: Implementations Support Additional Xdg Base Directory

These rules are ALWAYS ACTIVE for all configuration file loading operations, CLI handlers that initialize configuration subsystems, file system utilities that resolve user or system configuration paths, and configuration parsers that need to locate configuration files before reading them.

### Rules

- **R-XDG-001** MAY: Implementations MAY support additional XDG Base Directory variables (XDG_DATA_HOME, XDG_CACHE_HOME) for other resource types beyond configuration.
- **R-XDG-002** MUST: Centralize XDG path resolution logic in a shared utility module to ensure consistent behavior across CLI handlers, file system utilities, and configuration loaders.
- **R-XDG-003** MUST: Implement fallback logic that follows XDG Base Directory specification defaults when XDG_CONFIG_HOME is unset, typically defaulting to a subdirectory within the user home directory.
- **R-XDG-004** MUST: Log the resolved configuration directory path during initialization to aid debugging when users have custom XDG environment variable configurations.
- **R-XDG-005** MUST: Validate directory existence and permissions with informative error messages when configuration loading fails.
- **R-XDG-006** MUST: Ensure all configuration loading paths use the same centralized XDG resolution function to prevent inconsistent path resolution across modules.

### Verify

```bash
# Discover the project test suite and execute tests covering configuration loading with XDG_CONFIG_HOME set to a custom path
XDG_CONFIG_HOME=/tmp/custom-config npm test -- --grep "XDG_CONFIG_HOME.*custom"

# Discover the project test suite and execute tests covering configuration loading with XDG_CONFIG_HOME unset to verify fallback behavior
unset XDG_CONFIG_HOME && npm test -- --grep "XDG_CONFIG_HOME.*unset|fallback"

# Inspect the codebase for all modules accessing environment variables for configuration paths and verify they use the centralized XDG-aware resolution mechanism
grep -r "process\.env\.XDG_CONFIG_HOME" --include="*.js" --include="*.ts" | grep -v "node_modules" | grep -v "test"

# Verify no hardcoded configuration paths bypass XDG resolution
grep -r "\./config\|~/.config\|/etc/" --include="*.js" --include="*.ts" | grep -v "node_modules" | grep -v "test" | grep -v "XDG"
```

**Accept when:**
- All configuration loading tests pass with both XDG_CONFIG_HOME set and unset
- Configuration path resolution is consistent across CLI handlers, file system utilities, and configuration loading modules
- Error messages clearly indicate the resolved configuration directory path when configuration loading fails
- All configuration path resolution uses the centralized XDG-aware utility
- Integration tests detect no inconsistent configuration path resolution

<enforcement>
Code review MUST verify all configuration path resolution uses the centralized XDG-aware utility. Integration tests MUST cover configuration loading with various XDG environment variable configurations. Static analysis MUST detect direct hardcoded configuration paths that bypass XDG resolution. CI pipeline MUST fail if integration tests detect inconsistent configuration path resolution. Claude Code MUST NOT skip or defer verification.
</enforcement>
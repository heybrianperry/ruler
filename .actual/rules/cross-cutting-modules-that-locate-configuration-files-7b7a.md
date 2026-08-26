# Adopt XDG Base Directory Specification for Configuration File Location: Modules That Locate Configuration Files Use

These rules are ALWAYS ACTIVE for all modules that locate configuration files, including CLI handlers, file system utilities, and configuration parsers that resolve user or system configuration paths.

### Rules

- **R-XDG-001** MUST: All modules that locate configuration files MUST use a consistent XDG-aware path resolution mechanism rather than hardcoded paths.
- **R-XDG-002** MUST: Centralize XDG path resolution logic in a shared utility module to ensure consistent behavior across CLI handlers, file system utilities, and configuration loaders.
- **R-XDG-003** MUST: Implement fallback logic that follows XDG Base Directory specification defaults when XDG_CONFIG_HOME is unset, typically defaulting to a subdirectory within the user home directory.
- **R-XDG-004** MUST: Log the resolved configuration directory path during initialization to aid debugging when users have custom XDG environment variable configurations.
- **R-XDG-005** SHOULD: Validate directory existence and permissions with informative error messages when configuration loading fails.
- **R-XDG-006** SHOULD: Document platform-specific behavior for Windows and macOS where XDG Base Directory specification is not natively supported.

### Verify

```bash
# Discover the project test suite and execute tests covering configuration loading with XDG_CONFIG_HOME set to a custom path
# (Exact command depends on project's test runner; typically: npm test, pytest, or similar)

# Discover the project test suite and execute tests covering configuration loading with XDG_CONFIG_HOME unset to verify fallback behavior
# (Exact command depends on project's test runner)

# Inspect the codebase for all modules accessing environment variables for configuration paths and verify they use the centralized XDG-aware resolution mechanism
grep -r "process\.env\.XDG_CONFIG_HOME" --include="*.js" --include="*.ts" | grep -v node_modules

# Verify no hardcoded configuration paths bypass XDG resolution
grep -r "~/.config" --include="*.js" --include="*.ts" | grep -v node_modules | grep -v test | grep -v comment
```

**Accept when:**
- All configuration loading tests pass with both XDG_CONFIG_HOME set and unset
- Configuration path resolution is consistent across CLI handlers, file system utilities, and configuration loading modules
- Error messages clearly indicate the resolved configuration directory path when configuration loading fails
- All modules accessing environment variables for configuration paths use the centralized XDG-aware utility
- No hardcoded configuration paths are found that bypass XDG resolution

<enforcement>
Code review MUST verify all configuration path resolution uses the centralized XDG-aware utility. Integration tests MUST cover configuration loading with various XDG environment variable configurations. Static analysis MUST detect direct hardcoded configuration paths that bypass XDG resolution. CI pipeline MUST fail if integration tests detect inconsistent configuration path resolution. Claude Code MUST NOT skip or defer verification.
</enforcement>
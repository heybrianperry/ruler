# Adopt XDG Base Directory Specification for Configuration Path Resolution: Configuration Path Resolution Logic Handle Cases

These rules are ALWAYS ACTIVE for all configuration file discovery and loading operations, core infrastructure modules responsible for application initialization, deployment scripts, and user-facing configuration path specifications.

### Rules

- **R-XDG-001** MUST: Configuration path resolution logic MUST handle cases where XDG_CONFIG_HOME is undefined or points to inaccessible directories with appropriate error handling.

### Verify

```bash
# Discover and execute the project's configuration validation test suite to verify XDG environment variable handling
npm test -- --grep "XDG|config.*path"

# Inspect the core configuration modules to confirm process.env.XDG_CONFIG_HOME is accessed before fallback paths
grep -r "process\.env\.XDG_CONFIG_HOME" src/ --include="*.js" --include="*.ts"

# Run integration tests with XDG_CONFIG_HOME set to various values (valid path, invalid path, undefined) to verify error handling
XDG_CONFIG_HOME="/valid/path" npm test -- --grep "config"
XDG_CONFIG_HOME="/invalid/nonexistent/path" npm test -- --grep "config"
unset XDG_CONFIG_HOME && npm test -- --grep "config"
```

**Accept when:**
- Configuration loading successfully resolves paths from XDG_CONFIG_HOME when the environment variable is set
- Configuration loading provides appropriate fallback behavior when XDG_CONFIG_HOME is undefined or inaccessible
- Error messages include resolved configuration paths to aid debugging
- All configuration-related modules use the established configuration infrastructure instead of hardcoded paths
- Integration tests pass with XDG_CONFIG_HOME set to valid paths, invalid paths, and undefined

<enforcement>
Clause R-XDG-001 MUST be verified by code review of configuration-related modules, integration tests under different XDG_CONFIG_HOME settings, and static analysis to detect direct filesystem path construction that bypasses XDG resolution. Claude Code MUST NOT skip or defer verification.
</enforcement>
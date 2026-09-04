# Adopt XDG Base Directory Specification for Configuration Path Resolution: Configuration Path Resolution Respect Xdg Base

These rules are ALWAYS ACTIVE for all configuration file discovery, loading operations, and core infrastructure modules responsible for application initialization.

### Rules

- **R-XDG-001** MUST: Configuration path resolution MUST respect the XDG Base Directory specification by reading process.env.XDG_CONFIG_HOME as the primary configuration directory source.
- **R-XDG-002** MUST: When XDG_CONFIG_HOME is undefined, implement fallback logic according to XDG specification defaults to ensure consistent behavior across environments.
- **R-XDG-003** MUST: Configuration path resolution MUST occur early in application initialization before any configuration-dependent modules are loaded.
- **R-XDG-004** SHOULD: Provide a diagnostic mode or logging option that displays resolved configuration paths to aid troubleshooting.
- **R-XDG-005** SHOULD: Implement robust error handling with clear error messages when XDG_CONFIG_HOME points to inaccessible or non-existent directories.

### Verify

```bash
# Discover and execute the project's configuration validation test suite to verify XDG environment variable handling
npm test -- --grep "XDG|config.*path"

# Inspect the core configuration modules to confirm process.env.XDG_CONFIG_HOME is accessed before fallback paths
grep -r "process\.env\.XDG_CONFIG_HOME" src/ --include="*.js" --include="*.ts"

# Run integration tests with XDG_CONFIG_HOME set to various values
XDG_CONFIG_HOME="/valid/path" npm test
XDG_CONFIG_HOME="/invalid/nonexistent/path" npm test
unset XDG_CONFIG_HOME && npm test

# Verify no hardcoded configuration paths bypass XDG resolution
grep -r "__dirname\|process\.cwd()" src/ --include="*.js" --include="*.ts" | grep -v "XDG\|fallback" || echo "No direct path construction detected"
```

**Accept when:**
- Configuration loading successfully resolves paths from XDG_CONFIG_HOME when the environment variable is set
- Configuration loading provides appropriate fallback behavior when XDG_CONFIG_HOME is undefined or inaccessible
- Error messages include resolved configuration paths to aid debugging
- All configuration-related modules access process.env.XDG_CONFIG_HOME before constructing fallback paths
- Integration tests pass with XDG_CONFIG_HOME set to valid paths, invalid paths, and undefined

<enforcement>
Code review of configuration-related modules MUST verify XDG environment variable checks are present. Integration tests MUST verify configuration discovery behavior under different XDG_CONFIG_HOME settings. Static analysis MUST detect direct filesystem path construction that bypasses XDG resolution. Pull requests introducing hardcoded configuration paths MUST be rejected during code review. Claude Code MUST NOT skip or defer verification.
</enforcement>
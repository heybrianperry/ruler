# Adopt XDG Base Directory Specification for Configuration Path Resolution: Error Messages Related Configuration Directory Access

These rules are ALWAYS ACTIVE for all configuration file discovery, loading operations, and core infrastructure modules responsible for application initialization that access the XDG Base Directory specification.

### Rules

- **R-XDG-001** SHOULD: Error messages related to configuration directory access SHOULD include the resolved XDG path to aid debugging.

### Verify

```bash
# Discover and execute the project's configuration validation test suite to verify XDG environment variable handling
npm test -- --grep "XDG|config.*path"

# Inspect the core configuration modules to confirm process.env.XDG_CONFIG_HOME is accessed before fallback paths
grep -r "XDG_CONFIG_HOME" src/ --include="*.js" --include="*.ts"

# Run integration tests with XDG_CONFIG_HOME set to various values
XDG_CONFIG_HOME="/valid/path" npm test -- --grep "config.*discovery"
XDG_CONFIG_HOME="/invalid/nonexistent/path" npm test -- --grep "config.*discovery"
unset XDG_CONFIG_HOME && npm test -- --grep "config.*discovery"
```

**Accept when:**
- Configuration loading successfully resolves paths from XDG_CONFIG_HOME when the environment variable is set
- Configuration loading provides appropriate fallback behavior when XDG_CONFIG_HOME is undefined or inaccessible
- Error messages include resolved configuration paths to aid debugging
- All configuration-related modules use the established configuration infrastructure instead of hardcoded paths

<enforcement>
Clause R-XDG-001 verification is mandatory. Code review MUST confirm error messages include resolved XDG paths. Integration tests MUST verify configuration discovery under different XDG_CONFIG_HOME settings. Static analysis MUST detect direct filesystem path construction that bypasses XDG resolution.
</enforcement>
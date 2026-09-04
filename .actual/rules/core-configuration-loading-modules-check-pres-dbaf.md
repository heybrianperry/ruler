# Adopt XDG Base Directory Specification for Configuration Path Resolution: Configuration Loading Modules Check Presence Xdg

These rules are ALWAYS ACTIVE for all configuration loading modules and core infrastructure responsible for application initialization and configuration file discovery.

### Rules

- **R-XDG-001** MUST: Configuration loading modules MUST check for the presence of XDG_CONFIG_HOME environment variable before falling back to default configuration paths.

### Verify

```bash
# Discover and execute the project's configuration validation test suite to verify XDG environment variable handling
npm test -- --grep "XDG|config.*path"

# Inspect the core configuration modules to confirm process.env.XDG_CONFIG_HOME is accessed before fallback paths
grep -r "process\.env\.XDG_CONFIG_HOME" src/ --include="*.js" --include="*.ts"

# Run integration tests with XDG_CONFIG_HOME set to various values
XDG_CONFIG_HOME=/tmp/valid npm test -- --grep "config.*discovery"
XDG_CONFIG_HOME=/nonexistent npm test -- --grep "config.*fallback"
unset XDG_CONFIG_HOME && npm test -- --grep "config.*default"
```

**Accept when:**
- Configuration loading successfully resolves paths from XDG_CONFIG_HOME when the environment variable is set
- Configuration loading provides appropriate fallback behavior when XDG_CONFIG_HOME is undefined or inaccessible
- Error messages include resolved configuration paths to aid debugging
- All configuration-related modules use the established configuration infrastructure rather than hardcoded paths
- Integration tests pass with XDG_CONFIG_HOME set to valid paths, invalid paths, and undefined states

<enforcement>
Clause R-XDG-001 is mandatory. Code review MUST verify XDG environment variable checks are present in all configuration-related modules. Pull requests introducing hardcoded configuration paths that bypass XDG resolution MUST be rejected. Static analysis MUST detect direct filesystem path construction that bypasses XDG resolution.
</enforcement>
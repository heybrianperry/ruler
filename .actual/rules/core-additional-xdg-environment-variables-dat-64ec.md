# Adopt XDG Base Directory Specification for Configuration Path Resolution: Additional Xdg Environment Variables Data Home

These rules are ALWAYS ACTIVE for all configuration file discovery and loading operations, core infrastructure modules responsible for application initialization, deployment scripts, and user-facing configuration path specifications.

### Rules

- **R-XDG-001** MAY: Additional XDG environment variables (XDG_DATA_HOME, XDG_CACHE_HOME) MAY be adopted for other resource types following the same pattern as XDG_CONFIG_HOME.
- **R-XDG-002** MUST: When XDG_CONFIG_HOME is undefined, implement fallback logic according to XDG specification defaults to ensure consistent behavior across environments.
- **R-XDG-003** MUST: Configuration path resolution MUST occur early in application initialization before any configuration-dependent modules are loaded.
- **R-XDG-004** SHOULD: Provide a diagnostic mode or logging option that displays resolved configuration paths to aid troubleshooting.
- **R-XDG-005** MUST: Configuration modules that bypass XDG resolution MUST be refactored to use the established configuration infrastructure.
- **R-XDG-006** MUST: Pull requests introducing hardcoded configuration paths MUST be rejected during code review.
- **R-XDG-007** SHOULD: Platform-specific code that requires alternative configuration mechanisms SHOULD document the rationale and provide equivalent functionality.

### Verify

```bash
# Discover and execute the project's configuration validation test suite to verify XDG environment variable handling
npm test -- --grep "XDG|config.*path"

# Inspect the core configuration modules to confirm process.env.XDG_CONFIG_HOME is accessed before fallback paths
grep -r "process\.env\.XDG_CONFIG_HOME" src/ --include="*.js" --include="*.ts"

# Run integration tests with XDG_CONFIG_HOME set to various values
XDG_CONFIG_HOME="/valid/path" npm test -- --grep "config.*discovery"
XDG_CONFIG_HOME="/invalid/nonexistent/path" npm test -- --grep "config.*discovery"
unset XDG_CONFIG_HOME && npm test -- --grep "config.*discovery"
```

**Accept when:**
- Configuration loading successfully resolves paths from XDG_CONFIG_HOME when the environment variable is set
- Configuration loading provides appropriate fallback behavior when XDG_CONFIG_HOME is undefined or inaccessible
- Error messages include resolved configuration paths to aid debugging
- Code review confirms XDG environment variable checks are present in configuration-related modules
- Integration tests verify configuration discovery behavior under different XDG_CONFIG_HOME settings
- Static analysis detects no direct filesystem path construction that bypasses XDG resolution

<enforcement>
Clause MUST NOT skip or defer verification. All rules in this file are mandatory for code review and must be verified before merge.
</enforcement>
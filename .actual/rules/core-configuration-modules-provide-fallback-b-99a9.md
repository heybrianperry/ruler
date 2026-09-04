# Adopt XDG Base Directory Specification for Configuration Path Resolution: Configuration Modules Provide Fallback Behavior When

These rules are ALWAYS ACTIVE for all configuration file discovery and loading operations, core infrastructure modules responsible for application initialization, and any code that constructs or resolves configuration file paths.

### Rules

- **R-XDG-001** SHOULD: Configuration modules SHOULD provide fallback behavior when XDG_CONFIG_HOME is not set, following XDG specification defaults (typically `~/.config` on Unix-like systems).
- **R-XDG-002** MUST: Configuration path resolution MUST occur early in application initialization before any configuration-dependent modules are loaded.
- **R-XDG-003** MUST: Configuration modules MUST check `process.env.XDG_CONFIG_HOME` before constructing fallback paths.
- **R-XDG-004** SHOULD: Configuration discovery SHOULD provide a diagnostic mode or logging option that displays resolved configuration paths to aid troubleshooting.
- **R-XDG-005** MUST: Error handling MUST include resolved configuration paths in error messages to aid debugging when configuration loading fails.
- **R-XDG-006** MUST: Code MUST NOT bypass XDG resolution by constructing hardcoded filesystem paths directly.

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

# Verify no hardcoded configuration paths bypass XDG resolution
grep -r "\./config\|/etc/\|~/." src/ --include="*.js" --include="*.ts" | grep -v XDG | grep -v fallback || echo "No direct path construction detected"
```

**Accept when:**
- Configuration loading successfully resolves paths from XDG_CONFIG_HOME when the environment variable is set
- Configuration loading provides appropriate fallback behavior when XDG_CONFIG_HOME is undefined or inaccessible
- Error messages include resolved configuration paths to aid debugging
- All configuration-related modules access `process.env.XDG_CONFIG_HOME` before constructing fallback paths
- Integration tests pass with XDG_CONFIG_HOME set to valid paths, invalid paths, and undefined
- No hardcoded configuration paths are found that bypass XDG resolution

<enforcement>
Clause Code MUST NOT skip or defer verification. All rules in this file are mandatory for configuration-related code changes. Pull requests introducing hardcoded configuration paths or bypassing XDG resolution MUST be rejected during code review.
</enforcement>
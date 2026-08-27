# Adopt XDG Base Directory Specification for Configuration File Discovery: Modules That Resolve Configuration File Paths

These rules are ALWAYS ACTIVE for all modules that resolve configuration file paths, including CLI initialization, configuration loading operations, and filesystem utilities that determine where configuration files are stored.

### Rules

- **R-XDG-001** MUST: All modules that resolve configuration file paths MUST use a consistent mechanism to read XDG_CONFIG_HOME from the process environment.
- **R-XDG-002** MUST: Implement fallback logic that uses the XDG Base Directory specification default when XDG_CONFIG_HOME is not set, typically resolving to a subdirectory within the user's home directory.
- **R-XDG-003** MUST: Centralize XDG_CONFIG_HOME resolution in a single utility module to ensure consistent behavior across all configuration loading code paths.
- **R-XDG-004** SHOULD: Cache the resolved configuration directory path during application initialization to avoid repeated environment variable lookups and ensure consistent behavior throughout the application lifecycle.
- **R-XDG-005** MUST: Gracefully handle cases where XDG_CONFIG_HOME points to inaccessible or non-existent directories with appropriate error handling.

### Verify

```bash
# Discover and run the project's test suite covering configuration directory resolution
npm test -- --testPathPattern=config

# Run integration tests verifying configuration loading with various XDG_CONFIG_HOME states
XDG_CONFIG_HOME=/tmp/test-config npm test -- --testPathPattern=integration
XDG_CONFIG_HOME="" npm test -- --testPathPattern=integration
unset XDG_CONFIG_HOME && npm test -- --testPathPattern=integration

# Verify static analysis or linting rules enforce consistent environment variable access
npm run lint -- --rule xdg-config-home

# Search for direct filesystem path construction that bypasses XDG resolution
grep -r "process\.env\.HOME" src/ --include="*.js" --include="*.ts" | grep -v node_modules
grep -r "hardcoded.*config" src/ --include="*.js" --include="*.ts" | grep -v node_modules
```

**Accept when:**
- All configuration loading tests pass with XDG_CONFIG_HOME set to various valid and invalid values
- Configuration directory resolution produces consistent results across all modules that implement it
- Error handling gracefully manages cases where XDG_CONFIG_HOME points to inaccessible or non-existent directories
- No direct filesystem path construction bypasses XDG resolution in configuration-related modules
- Static analysis confirms consistent environment variable access patterns across configuration code paths

<enforcement>
Clause MUST NOT skip or defer verification. Code review MUST verify that new configuration loading code consults XDG_CONFIG_HOME. Automated tests MUST cover configuration discovery with various environment variable states. Static analysis MUST detect direct filesystem path construction that bypasses XDG resolution. Violations require refactoring to use XDG-based configuration discovery and documentation of any approved exceptions.
</enforcement>
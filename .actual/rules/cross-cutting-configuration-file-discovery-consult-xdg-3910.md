# Adopt XDG Base Directory Specification for Configuration File Discovery: Configuration File Discovery Consult Xdg Config

These rules are ALWAYS ACTIVE for all configuration file discovery and loading operations, CLI initialization that requires access to user or system configuration, filesystem utilities that resolve global or user-specific configuration directories, and any module that needs to determine where configuration files are stored.

### Rules

- **R-XDG-001** MUST: Configuration file discovery MUST consult the XDG_CONFIG_HOME environment variable as the primary source for determining the user configuration directory location.
- **R-XDG-002** MUST: Implement fallback logic that uses the XDG Base Directory specification default when XDG_CONFIG_HOME is not set, typically resolving to a subdirectory within the user's home directory.
- **R-XDG-003** MUST: Centralize XDG_CONFIG_HOME resolution in a single utility module to ensure consistent behavior across all configuration loading code paths.
- **R-XDG-004** SHOULD: Cache the resolved configuration directory path during application initialization to avoid repeated environment variable lookups and ensure consistent behavior throughout the application lifecycle.
- **R-XDG-005** MUST: Resolve configuration directory location during initialization and cache the result for the application lifecycle to prevent inconsistent configuration resolution if XDG_CONFIG_HOME changes during runtime.

### Verify

```bash
# Discover the project's test suite and execute tests covering configuration directory resolution
npm test -- --testPathPattern=config.*discovery

# Run integration tests verifying configuration loading with various XDG_CONFIG_HOME values
XDG_CONFIG_HOME=/tmp/test-config npm test -- --testPathPattern=config.*integration

# Verify configuration loading when XDG_CONFIG_HOME is unset
unset XDG_CONFIG_HOME && npm test -- --testPathPattern=config.*fallback

# Verify configuration loading when XDG_CONFIG_HOME points to non-existent directory
XDG_CONFIG_HOME=/nonexistent/path npm test -- --testPathPattern=config.*error

# Identify and run static analysis or linting rules enforcing consistent environment variable access
npm run lint -- --rule="no-hardcoded-paths"
```

**Accept when:**
- All configuration loading tests pass with XDG_CONFIG_HOME set to various valid and invalid values
- Configuration directory resolution produces consistent results across all modules that implement it
- Error handling gracefully manages cases where XDG_CONFIG_HOME points to inaccessible or non-existent directories
- Code review verifies that new configuration loading code consults XDG_CONFIG_HOME
- Static analysis detects and flags direct filesystem path construction that bypasses XDG resolution

<enforcement>
Clause MUST NOT skip or defer verification. All configuration file discovery code MUST be reviewed for XDG_CONFIG_HOME compliance before merge. Violations require refactoring or documented exception approval.
</enforcement>
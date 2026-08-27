# Adopt XDG Base Directory Specification for Configuration File Discovery: Configuration Loading Logic Implement Fallback Behavior

These rules are ALWAYS ACTIVE for all configuration file discovery and loading operations, CLI initialization that requires access to user or system configuration, filesystem utilities that resolve global or user-specific configuration directories, and any module that needs to determine where configuration files are stored.

### Rules

- **R-XDG-001** SHOULD: Configuration loading logic SHOULD implement fallback behavior when XDG_CONFIG_HOME is not set, following XDG Base Directory specification defaults.
- **R-XDG-002** MUST: Centralize XDG_CONFIG_HOME resolution in a single utility module to ensure consistent behavior across all configuration loading code paths.
- **R-XDG-003** SHOULD: Cache the resolved configuration directory path during application initialization to avoid repeated environment variable lookups and ensure consistent behavior throughout the application lifecycle.
- **R-XDG-004** MUST: Implement fallback logic that uses the XDG Base Directory specification default when XDG_CONFIG_HOME is not set, typically resolving to a subdirectory within the user's home directory.
- **R-XDG-005** MUST: Verify that new configuration loading code consults XDG_CONFIG_HOME before constructing filesystem paths.
- **R-XDG-006** MUST: Implement error handling that gracefully manages cases where XDG_CONFIG_HOME points to inaccessible or non-existent directories.

### Verify

```bash
# Discover and run the project's test suite covering configuration directory resolution
npm test -- --testPathPattern=config.*xdg|xdg.*config

# Run integration tests verifying configuration loading with various XDG_CONFIG_HOME states
XDG_CONFIG_HOME=/tmp/test-config npm test -- --testPathPattern=integration.*config
XDG_CONFIG_HOME="" npm test -- --testPathPattern=integration.*config
unset XDG_CONFIG_HOME && npm test -- --testPathPattern=integration.*config

# Identify and run static analysis or linting rules for environment variable access patterns
npm run lint -- --rule=no-process-env-direct

# Verify configuration directory resolution consistency across modules
grep -r "XDG_CONFIG_HOME" src/ --include="*.js" --include="*.ts" | wc -l
```

**Accept when:**
- All configuration loading tests pass with XDG_CONFIG_HOME set to various valid and invalid values
- Configuration directory resolution produces consistent results across all modules that implement it
- Error handling gracefully manages cases where XDG_CONFIG_HOME points to inaccessible or non-existent directories
- Code review verifies that new configuration loading code consults XDG_CONFIG_HOME
- Static analysis detects and flags direct filesystem path construction that bypasses XDG resolution
- Integration tests confirm configuration discovery behavior when XDG_CONFIG_HOME is set, unset, or points to non-existent directories

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration loading code MUST be reviewed against these rules before merge. Violations require refactoring to use XDG-based configuration discovery or documented exceptions approved by the architecture review team.
</enforcement>
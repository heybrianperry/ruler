# Adopt XDG Base Directory Specification for Configuration File Discovery: Configuration Directory Resolution Occur During Application

These rules are ALWAYS ACTIVE for all configuration file discovery and loading operations, CLI initialization that requires access to user or system configuration, filesystem utilities that resolve global or user-specific configuration directories, and any module that needs to determine where configuration files are stored.

### Rules

- **R-XDG-001** SHOULD: Configuration directory resolution SHOULD occur during application initialization to fail fast if the environment is misconfigured.
- **R-XDG-002** MUST: Centralize XDG_CONFIG_HOME resolution in a single utility module to ensure consistent behavior across all configuration loading code paths.
- **R-XDG-003** MUST: Implement fallback logic that uses the XDG Base Directory specification default when XDG_CONFIG_HOME is not set, typically resolving to a subdirectory within the user's home directory.
- **R-XDG-004** SHOULD: Cache the resolved configuration directory path during application initialization to avoid repeated environment variable lookups and ensure consistent behavior throughout the application lifecycle.
- **R-XDG-005** MUST: All configuration loading code paths MUST consult XDG_CONFIG_HOME or its fallback default; direct filesystem path construction that bypasses XDG resolution is prohibited.
- **R-XDG-006** MUST: Error handling MUST gracefully manage cases where XDG_CONFIG_HOME points to inaccessible or non-existent directories.

### Verify

```bash
# Discover and run the project's test suite covering configuration directory resolution
npm test -- --testPathPattern="config|xdg" --verbose

# Run integration tests with various XDG_CONFIG_HOME values
XDG_CONFIG_HOME=/tmp/test-config npm test -- --testPathPattern="integration.*config"
XDG_CONFIG_HOME="" npm test -- --testPathPattern="integration.*config"
unset XDG_CONFIG_HOME && npm test -- --testPathPattern="integration.*config"

# Verify configuration loading behavior with non-existent directories
XDG_CONFIG_HOME=/nonexistent/path npm test -- --testPathPattern="config.*error|fallback"

# Run static analysis to detect direct filesystem path construction
grep -r "process\.cwd()\|__dirname\|path\.join.*config" src/ --include="*.ts" --include="*.js" | grep -v "node_modules" | grep -v ".actual"

# Verify consistent XDG_CONFIG_HOME access patterns across modules
grep -r "XDG_CONFIG_HOME" src/ --include="*.ts" --include="*.js" | wc -l
```

**Accept when:**
- All configuration loading tests pass with XDG_CONFIG_HOME set to various valid and invalid values
- Configuration directory resolution produces consistent results across all modules that implement it
- Error handling gracefully manages cases where XDG_CONFIG_HOME points to inaccessible or non-existent directories
- Static analysis detects no direct filesystem path construction that bypasses XDG resolution in configuration-related code
- Configuration directory resolution occurs during application initialization, not lazily at runtime

<enforcement>
Clause MUST NOT skip or defer verification. Code review MUST verify that new configuration loading code consults XDG_CONFIG_HOME. Automated tests MUST cover configuration discovery with various environment variable states. Static analysis MUST detect direct filesystem path construction that bypasses XDG resolution. Violations require code review feedback, test failures, or documentation updates for approved exceptions.
</enforcement>
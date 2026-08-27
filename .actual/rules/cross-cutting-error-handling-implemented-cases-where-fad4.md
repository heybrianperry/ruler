# Adopt XDG Base Directory Specification for Configuration File Discovery: Error Handling Implemented Cases Where Resolved

These rules are ALWAYS ACTIVE for all configuration file discovery and loading operations, CLI initialization that requires access to user or system configuration, filesystem utilities that resolve global or user-specific configuration directories, and any module that needs to determine where configuration files are stored.

### Rules

- **R-XDG-001** MUST: Error handling MUST be implemented for cases where the resolved configuration directory is inaccessible or does not exist.

### Verify

```bash
# Discover the project's test suite and execute tests covering configuration directory resolution with various XDG_CONFIG_HOME values
npm test -- --testPathPattern="config|xdg" 2>&1 | head -50

# Locate and run integration tests that verify configuration loading behavior when XDG_CONFIG_HOME is set, unset, or points to non-existent directories
XDG_CONFIG_HOME="/nonexistent" npm test -- --testPathPattern="config" 2>&1 | head -50

# Identify static analysis or linting rules that enforce consistent environment variable access patterns across configuration-related modules
npm run lint -- --format=json 2>&1 | grep -i "xdg\|config" | head -20
```

**Accept when:**
- All configuration loading tests pass with XDG_CONFIG_HOME set to various valid and invalid values
- Configuration directory resolution produces consistent results across all modules that implement it
- Error handling gracefully manages cases where XDG_CONFIG_HOME points to inaccessible or non-existent directories

<enforcement>
Clause R-XDG-001 is mandatory. Code review MUST verify that new configuration loading code implements error handling for inaccessible or non-existent configuration directories. Automated tests MUST cover configuration discovery with various environment variable states. Static analysis MUST detect direct filesystem path construction that bypasses XDG resolution. Violations require refactoring to use XDG-based configuration discovery with proper error handling, test failures when configuration loading does not respect XDG_CONFIG_HOME, and documentation updates for any approved exceptions.
</enforcement>
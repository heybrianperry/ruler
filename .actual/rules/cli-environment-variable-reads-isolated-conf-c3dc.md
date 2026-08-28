# Adopt XDG Base Directory Specification for Configuration Discovery via process.env: Environment Variable Reads Isolated Configuration Discovery

These rules are ALWAYS ACTIVE for CLI handler modules that require user configuration file access, configuration discovery and initialization code paths, and Unix-like operating systems where XDG Base Directory specification is applicable.

### Rules

- **R-XDG-001** SHOULD: Environment variable reads SHOULD be isolated to configuration discovery modules to maintain separation of concerns.
- **R-XDG-002** MUST: Configuration discovery code MUST read `XDG_CONFIG_HOME` from `process.env` and use the value as the base configuration directory.
- **R-XDG-003** MUST: Fallback logic MUST provide XDG-compliant default path (`$HOME/.config`) when `XDG_CONFIG_HOME` is undefined.
- **R-XDG-004** MUST: Configuration path construction MUST use the runtime's path module to ensure proper path separator handling and normalization across different Unix-like systems.
- **R-XDG-005** SHOULD: Configuration directory path resolution SHOULD be cached after first access to avoid repeated environment variable reads and path resolution operations.
- **R-XDG-006** MUST: Directory existence and permissions MUST be validated before reading configuration files from the resolved path.

### Verify

```bash
# Locate the project's test suite and identify tests covering configuration discovery
find . -type f -name '*.test.ts' -o -name '*.spec.ts' | xargs grep -l 'XDG_CONFIG_HOME\|configuration.*discovery' || echo "No configuration discovery tests found"

# Inspect the CLI handler module to confirm process.env.XDG_CONFIG_HOME access pattern
grep -r 'process\.env\.XDG_CONFIG_HOME' src/cli/ || echo "XDG_CONFIG_HOME access not found in CLI handlers"

# Verify fallback logic implementation
grep -r 'XDG_CONFIG_HOME.*||\|XDG_CONFIG_HOME.*??\|fallback' src/cli/ || echo "Fallback logic not found"

# Set XDG_CONFIG_HOME to a test directory and verify CLI locates configuration files
XDG_CONFIG_HOME=/tmp/test-config npm test -- --testNamePattern='XDG_CONFIG_HOME.*set' || echo "Test with XDG_CONFIG_HOME set failed"

# Unset XDG_CONFIG_HOME and verify fallback behavior
unset XDG_CONFIG_HOME && npm test -- --testNamePattern='XDG_CONFIG_HOME.*unset\|fallback' || echo "Test with XDG_CONFIG_HOME unset failed"
```

**Accept when:**
- Configuration discovery code reads `XDG_CONFIG_HOME` from `process.env` and uses the value as the base configuration directory
- Fallback logic provides XDG-compliant default path when `XDG_CONFIG_HOME` is undefined
- Tests verify configuration discovery behavior with both set and unset `XDG_CONFIG_HOME` environment variable states
- Path module is used for path construction and normalization
- Directory existence and permissions are validated before configuration file access
- Configuration path resolution is cached to avoid repeated lookups

<enforcement>
Code review MUST verify that configuration discovery uses `process.env.XDG_CONFIG_HOME`. Unit tests MUST cover XDG_CONFIG_HOME presence and absence scenarios. Integration tests MUST validate configuration file discovery in XDG-compliant directory structure. Code review MUST reject configuration discovery code that bypasses `XDG_CONFIG_HOME`. Test failures MUST be addressed when configuration discovery does not respect environment variable settings. Platform-specific exceptions require architectural review approval and must be documented in code comments.
</enforcement>
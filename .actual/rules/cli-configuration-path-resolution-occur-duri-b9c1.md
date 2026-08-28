# Adopt XDG Base Directory Specification for Configuration Discovery via process.env: Configuration Path Resolution Occur During Cli

These rules are ALWAYS ACTIVE for CLI handler modules that require user configuration file access, configuration discovery and initialization code paths, and Unix-like operating systems where XDG Base Directory specification is applicable.

### Rules

- **R-XDG-001** SHOULD: Configuration path resolution SHOULD occur during CLI handler initialization before command processing begins.
- **R-XDG-002** MUST: Configuration discovery code MUST read XDG_CONFIG_HOME from process.env and use the value as the base configuration directory.
- **R-XDG-003** MUST: Fallback logic MUST provide XDG-compliant default path ($HOME/.config) when XDG_CONFIG_HOME is undefined.
- **R-XDG-004** MUST: Path construction MUST use the runtime's path module to ensure proper path separator handling and normalization across different Unix-like systems.
- **R-XDG-005** SHOULD: Configuration directory path resolution SHOULD be cached after first access to avoid repeated environment variable reads and path resolution operations.
- **R-XDG-006** MUST: Directory existence and permissions MUST be validated before reading configuration files from the resolved path.
- **R-XDG-007** MUST: Platform-specific configuration discovery (Windows, macOS) MUST use alternative environment variables or conventions with architectural review approval when XDG specification does not apply.

### Verify

```bash
# Locate the project's test suite and identify tests covering configuration discovery
find . -type f -name '*.test.*' -o -name '*.spec.*' | grep -i config

# Execute tests that verify XDG_CONFIG_HOME environment variable handling
npm test -- --grep "XDG_CONFIG_HOME"

# Inspect the CLI handler module to confirm process.env.XDG_CONFIG_HOME access pattern
grep -r "process\.env\.XDG_CONFIG_HOME" src/cli/

# Set XDG_CONFIG_HOME to a test directory and verify CLI locates configuration files
XDG_CONFIG_HOME=/tmp/test-config npm run cli -- --help

# Unset XDG_CONFIG_HOME and verify fallback behavior
unset XDG_CONFIG_HOME && npm run cli -- --help

# Verify fallback logic provides default path
grep -r "HOME.*\.config" src/cli/
```

**Accept when:**
- Configuration discovery code reads XDG_CONFIG_HOME from process.env and uses the value as the base configuration directory
- Fallback logic provides XDG-compliant default path when XDG_CONFIG_HOME is undefined
- Tests verify configuration discovery behavior with both set and unset XDG_CONFIG_HOME environment variable states
- Path module is used for path construction and normalization
- Directory existence and permissions are validated before configuration file access
- Configuration path resolution occurs during CLI handler initialization before command processing

<enforcement>
Clause Code MUST NOT skip or defer verification. Configuration discovery code that bypasses XDG_CONFIG_HOME will be rejected in code review. Test failures when configuration discovery does not respect environment variable settings will block merge. Documentation updates are required when platform-specific exceptions are introduced.
</enforcement>
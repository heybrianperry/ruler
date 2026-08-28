# Adopt XDG Base Directory Specification for Configuration Discovery via process.env: Cli Handlers Access Environment Variables Through

These rules are ALWAYS ACTIVE for CLI handler modules that require user configuration file access, configuration discovery and initialization code paths, and all Unix-like operating system targets where XDG Base Directory specification is applicable.

### Rules

- **R-XDG-001** MUST: CLI handlers MUST access environment variables through the Node.js process.env API rather than alternative configuration sources when discovering user configuration files.
- **R-XDG-002** MUST: Configuration discovery code MUST read XDG_CONFIG_HOME from process.env and use the value as the base configuration directory.
- **R-XDG-003** MUST: Fallback logic MUST provide XDG-compliant default path ($HOME/.config) when XDG_CONFIG_HOME is undefined, following XDG Base Directory specification version 0.8.
- **R-XDG-004** SHOULD: Configuration path construction SHOULD use the runtime's path module to ensure proper path separator handling and normalization across different Unix-like systems.
- **R-XDG-005** SHOULD: The resolved configuration directory path SHOULD be cached after first access to avoid repeated environment variable reads and path resolution operations.
- **R-XDG-006** MUST: Directory permissions MUST be validated before reading configuration files to prevent security implications from untrusted or world-writable directories.
- **R-XDG-007** MUST: Platform-specific configuration discovery (Windows, macOS) MUST use alternative environment variables or conventions only with architectural review approval and documented exceptions.

### Verify

```bash
# Locate the project's test suite and identify tests covering configuration discovery
find . -type f -name '*.test.ts' -o -name '*.spec.ts' | xargs grep -l 'XDG_CONFIG_HOME\|configuration.*discovery' || echo "No configuration discovery tests found"

# Inspect the CLI handler module to confirm process.env.XDG_CONFIG_HOME access pattern
grep -r 'process\.env\.XDG_CONFIG_HOME' src/cli/handlers.ts || echo "XDG_CONFIG_HOME access not found"

# Verify fallback logic implementation
grep -A 5 'XDG_CONFIG_HOME' src/cli/handlers.ts | grep -E '\$HOME|\.config|fallback' || echo "Fallback logic not verified"

# Test with XDG_CONFIG_HOME set to a test directory
XDG_CONFIG_HOME=/tmp/test-config npm test -- --testNamePattern='XDG.*set' || echo "Test with XDG_CONFIG_HOME set failed"

# Test with XDG_CONFIG_HOME unset to verify fallback behavior
unset XDG_CONFIG_HOME && npm test -- --testNamePattern='XDG.*unset|fallback' || echo "Test with XDG_CONFIG_HOME unset failed"
```

**Accept when:**
- Configuration discovery code reads XDG_CONFIG_HOME from process.env and uses the value as the base configuration directory
- Fallback logic provides XDG-compliant default path when XDG_CONFIG_HOME is undefined
- Tests verify configuration discovery behavior with both set and unset XDG_CONFIG_HOME environment variable states
- Directory permission validation is implemented before reading configuration files
- Path module is used for path construction and normalization
- Configuration directory path resolution is cached to avoid repeated lookups
- Platform-specific exceptions are documented in code comments with rationale

<enforcement>
Clause Code MUST NOT skip or defer verification. All rules R-XDG-001 through R-XDG-007 are mandatory for configuration discovery code paths. Violations must be caught during code review and rejected until compliant.
</enforcement>
# Adopt XDG Base Directory Specification for Configuration Discovery via process.env: Configuration Discovery Code Provide Fallback Mechanism

These rules are ALWAYS ACTIVE for CLI handler modules that require user configuration file access, configuration discovery and initialization code paths, and Unix-like operating systems where XDG Base Directory specification is applicable.

### Rules

- **R-XDG-001** MUST: Configuration discovery code MUST provide a fallback mechanism when XDG_CONFIG_HOME is undefined or empty.
- **R-XDG-002** MUST: Configuration discovery code MUST read XDG_CONFIG_HOME from process.env and use the value as the base configuration directory.
- **R-XDG-003** MUST: Fallback logic MUST provide XDG-compliant default path ($HOME/.config) when XDG_CONFIG_HOME is undefined, per XDG Base Directory specification version 0.8.
- **R-XDG-004** SHOULD: Configuration path construction SHOULD use the runtime's path module to ensure proper path separator handling and normalization across different Unix-like systems.
- **R-XDG-005** SHOULD: Consider caching the resolved configuration directory path after first access to avoid repeated environment variable reads and path resolution operations.
- **R-XDG-006** MUST: Platform-specific configuration discovery (Windows, macOS) using alternative environment variables or conventions MUST be documented in code comments with rationale and approved by configuration architecture owner.

### Verify

```bash
# Locate the project's test suite and identify tests covering configuration discovery
find . -type f -name '*.test.*' -o -name '*.spec.*' | xargs grep -l 'XDG_CONFIG_HOME\|configuration.*discovery' || echo "No tests found"

# Inspect the CLI handler module to confirm process.env.XDG_CONFIG_HOME access pattern
grep -r 'process\.env\.XDG_CONFIG_HOME' src/cli/ || echo "XDG_CONFIG_HOME access not found"

# Verify fallback logic implementation
grep -r 'XDG_CONFIG_HOME.*||\|XDG_CONFIG_HOME.*??\|fallback' src/cli/ || echo "Fallback logic not found"

# Set XDG_CONFIG_HOME to a test directory and verify CLI locates configuration files
XDG_CONFIG_HOME=/tmp/test-config npm test -- --testNamePattern='XDG_CONFIG_HOME.*set' || echo "Test with XDG_CONFIG_HOME set failed"

# Unset XDG_CONFIG_HOME and verify fallback behavior
unset XDG_CONFIG_HOME && npm test -- --testNamePattern='XDG_CONFIG_HOME.*unset\|fallback' || echo "Test with XDG_CONFIG_HOME unset failed"
```

**Accept when:**
- Configuration discovery code reads XDG_CONFIG_HOME from process.env and uses the value as the base configuration directory
- Fallback logic provides XDG-compliant default path when XDG_CONFIG_HOME is undefined
- Tests verify configuration discovery behavior with both set and unset XDG_CONFIG_HOME environment variable states
- Configuration path construction uses the runtime's path module for proper separator handling
- Platform-specific exceptions are documented in code comments with architectural review approval

<enforcement>
Code review verification that configuration discovery uses process.env.XDG_CONFIG_HOME is mandatory. Unit tests covering XDG_CONFIG_HOME presence and absence scenarios are mandatory. Integration tests validating configuration file discovery in XDG-compliant directory structure are mandatory. Claude Code MUST NOT skip or defer verification.
</enforcement>
# Adopt XDG Base Directory Specification for Configuration Discovery via process.env: Configuration File Discovery Read Xdg Config

These rules are ALWAYS ACTIVE for CLI handler modules that require user configuration file access, configuration discovery and initialization code paths, and all Unix-like operating system targets where XDG Base Directory specification is applicable.

### Rules

- **R-XDG-001** MUST: Configuration file discovery MUST read the XDG_CONFIG_HOME environment variable via process.env to determine the base configuration directory location.
- **R-XDG-002** MUST: Implement fallback logic to default to $HOME/.config (XDG-compliant default path per XDG Base Directory specification version 0.8) when XDG_CONFIG_HOME is unset.
- **R-XDG-003** MUST: Use the runtime's path module to ensure proper path separator handling and normalization across different Unix-like systems.
- **R-XDG-004** SHOULD: Cache the resolved configuration directory path after first access to avoid repeated environment variable reads and path resolution operations.
- **R-XDG-005** MUST: Validate directory existence before attempting to read configuration files from the resolved path.
- **R-XDG-006** MUST: Validate directory permissions before reading configuration files; document security expectations for configuration directory ownership.
- **R-XDG-007** MUST: Implement platform detection and provide Windows-specific configuration discovery path when applicable; document platform-specific behavior in user-facing documentation.

### Verify

```bash
# Locate the project's test suite and identify tests covering configuration discovery
find . -type f -name '*.test.*' -o -name '*.spec.*' | grep -i config | head -20

# Execute tests that verify XDG_CONFIG_HOME environment variable handling
npm test -- --testNamePattern="XDG|xdg|config.*discovery" 2>&1 | head -50

# Inspect the CLI handler module to confirm process.env.XDG_CONFIG_HOME access pattern
grep -r "process\.env\.XDG_CONFIG_HOME" src/ --include="*.ts" --include="*.js"

# Verify fallback logic implementation
grep -r "XDG_CONFIG_HOME\|HOME.*\.config" src/ --include="*.ts" --include="*.js" -A 3 -B 3

# Set XDG_CONFIG_HOME to a test directory and verify CLI locates configuration files
XDG_CONFIG_HOME=/tmp/test-xdg npm run cli -- --help 2>&1 | head -20

# Unset XDG_CONFIG_HOME and verify fallback behavior
unset XDG_CONFIG_HOME && npm run cli -- --help 2>&1 | head -20

# Verify configuration discovery tests exist and pass
npm test -- --testNamePattern="config.*discovery|xdg" --verbose 2>&1
```

**Accept when:**
- Configuration discovery code reads XDG_CONFIG_HOME from process.env and uses the value as the base configuration directory
- Fallback logic provides XDG-compliant default path ($HOME/.config) when XDG_CONFIG_HOME is undefined
- Tests verify configuration discovery behavior with both set and unset XDG_CONFIG_HOME environment variable states
- Directory existence and permission validation is performed before reading configuration files
- Path module is used for path construction and normalization
- Platform-specific configuration discovery is implemented with appropriate documentation
- Code review confirms no hardcoded filesystem paths bypass XDG_CONFIG_HOME discovery

<enforcement>
Clause Code MUST NOT skip or defer verification. All rules R-XDG-001 through R-XDG-007 are mandatory and must be verified before accepting configuration discovery implementation.
</enforcement>
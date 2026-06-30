# Isolate Environment Variable Access for Configuration Discovery: Configuration Resolution Logic

These rules are ALWAYS ACTIVE for all runtime configuration derived from environment variables, including XDG Base Directory specification variables, configuration path resolution in core utilities and CLI handlers, and any module that currently accesses process.env directly for configuration purposes.

### Rules

- **R-CONFIG-001** SHOULD: Configuration resolution logic SHOULD be testable without requiring actual environment variables to be set.
- **R-CONFIG-002** MUST: All runtime configuration derived from environment variables MUST be accessed through a dedicated configuration module, not directly via process.env.
- **R-CONFIG-003** MUST: Configuration consumers MUST receive configuration through dependency injection or function parameters, not by accessing process.env directly.
- **R-CONFIG-004** SHOULD: Configuration module SHOULD provide typed configuration accessors (e.g., getXdgConfigHome()) with validation and clear error messages.
- **R-CONFIG-005** MUST: Direct process.env access outside the dedicated configuration module MUST be prohibited by linting rules and CI checks.

### Verify

```bash
# Check for direct XDG_CONFIG_HOME access outside config module
grep -r 'process\.env\.XDG_CONFIG_HOME' src/ --exclude-dir=config --exclude-dir=__tests__

# Check for any direct process.env access outside config module
grep -r 'process\.env\.' src/ | grep -v 'src/config/' | grep -v '__tests__' | grep -v 'NODE_ENV'

# Run configuration module tests with coverage
npm test -- --testPathPattern=config --coverage
```

**Accept when:**
- No direct process.env.XDG_CONFIG_HOME access exists outside the dedicated configuration module
- All configuration consumers receive configuration through dependency injection or parameters
- Configuration module has >90% test coverage with mocked environment variable scenarios
- All refactored modules pass unit tests verifying configuration injection patterns

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep-based checks MUST pass, test coverage MUST exceed 90%, and code review MUST confirm dependency injection patterns before accepting changes.
</enforcement>
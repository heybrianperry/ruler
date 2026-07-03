# Isolate Environment Variable Access for Configuration Discovery: Environment Variable Names

These rules are ALWAYS ACTIVE for all runtime code that accesses environment variables for configuration discovery, including core utilities, CLI handlers, and any module that currently accesses process.env directly.

### Rules

- **R-ENV-001** SHOULD: Environment variable names SHOULD be documented in a central configuration schema or constants file.

### Verify

```bash
# Check for direct process.env.XDG_CONFIG_HOME access outside config module
grep -r 'process\.env\.XDG_CONFIG_HOME' src/ --exclude-dir=config --exclude-dir=__tests__

# Check for any direct process.env access outside config module (excluding NODE_ENV)
grep -r 'process\.env\.' src/ | grep -v 'src/config/' | grep -v '__tests__' | grep -v 'NODE_ENV'

# Run configuration module tests with coverage
npm test -- --testPathPattern=config --coverage
```

**Accept when:**
- No direct process.env.XDG_CONFIG_HOME access exists outside the dedicated configuration module
- All configuration consumers receive configuration through dependency injection or parameters
- Configuration module has >90% test coverage with mocked environment variable scenarios
- A dedicated src/config module exists that exports typed configuration accessors (e.g., getXdgConfigHome())
- src/core/FileSystemUtils.ts and src/cli/handlers.ts receive configuration through constructor injection or function parameters
- Configuration validation is implemented with clear error messages for missing or invalid environment variables

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep-based checks MUST pass, test coverage MUST exceed 90%, and configuration injection patterns MUST be verified before accepting this rule as satisfied.
</enforcement>
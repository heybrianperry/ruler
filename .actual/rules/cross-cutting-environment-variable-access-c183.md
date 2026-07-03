# Isolate Environment Variable Access for Configuration Discovery: Environment Variable Access

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-ENV-001** MUST: All environment variable access for configuration discovery MUST be isolated to a dedicated configuration module

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
- All refactored modules (src/core/FileSystemUtils.ts, src/cli/handlers.ts) receive configuration through constructor injection or function parameters

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes that touch environment variable access patterns.
</enforcement>
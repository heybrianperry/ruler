# Isolate Environment Variable Access for Configuration Discovery: Configuration Modules Implement

These rules are ALWAYS ACTIVE for all runtime configuration derived from environment variables, including XDG_CONFIG_HOME and other XDG Base Directory specification variables, configuration path resolution in core utilities and CLI handlers, and any module that currently accesses process.env directly for configuration purposes.

### Rules

- **R-CONFIG-001** MAY: Configuration modules MAY implement caching to avoid repeated process.env lookups.

### Verify

```bash
# Detect direct process.env.XDG_CONFIG_HOME access outside configuration module
grep -r 'process\.env\.XDG_CONFIG_HOME' src/ --exclude-dir=config --exclude-dir=__tests__

# Detect all direct process.env access outside configuration module (excluding NODE_ENV and system variables)
grep -r 'process\.env\.' src/ | grep -v 'src/config/' | grep -v '__tests__' | grep -v 'NODE_ENV'

# Run configuration module tests with coverage
npm test -- --testPathPattern=config --coverage
```

**Accept when:**
- No direct process.env.XDG_CONFIG_HOME access exists outside the dedicated configuration module
- All configuration consumers receive configuration through dependency injection or parameters
- Configuration module has >90% test coverage with mocked environment variable scenarios
- ESLint rules prohibit direct process.env access outside allowed locations
- All refactored modules pass unit tests verifying configuration injection

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep-based checks MUST pass before accepting changes. Configuration module test coverage MUST meet the >90% threshold. Code review MUST verify dependency injection patterns are correctly implemented.
</enforcement>
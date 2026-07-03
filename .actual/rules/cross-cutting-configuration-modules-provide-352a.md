# Isolate Environment Variable Access for Configuration Discovery: Configuration Modules Provide

These rules are ALWAYS ACTIVE for all runtime code that accesses environment variables for configuration discovery, including core utilities, CLI handlers, and any module deriving configuration from process.env.

### Rules

- **R-CONFIG-001** MUST: Configuration modules MUST provide type-safe accessors for environment variables with validation and default values.
- **R-CONFIG-002** MUST: All runtime configuration derived from environment variables MUST be accessed through centralized configuration module accessors, not direct process.env access.
- **R-CONFIG-003** MUST: XDG_CONFIG_HOME and other XDG Base Directory specification variables MUST be accessed only through configuration module type-safe accessors.
- **R-CONFIG-004** MUST: Configuration path resolution in core utilities and CLI handlers MUST receive configuration through dependency injection or function parameters, not direct process.env access.
- **R-CONFIG-005** SHOULD: Configuration module accessors SHOULD implement validation with clear error messages for missing or invalid environment variables.
- **R-CONFIG-006** SHOULD: Configuration module SHOULD cache parsed configuration values to avoid repeated environment variable access.

### Verify

```bash
# Detect direct process.env.XDG_CONFIG_HOME access outside configuration module
grep -r 'process\.env\.XDG_CONFIG_HOME' src/ --exclude-dir=config --exclude-dir=__tests__

# Detect all direct process.env access outside configuration module (excluding NODE_ENV and system variables)
grep -r 'process\.env\.' src/ | grep -v 'src/config/' | grep -v '__tests__' | grep -v 'NODE_ENV'

# Verify configuration module test coverage
npm test -- --testPathPattern=config --coverage
```

**Accept when:**
- No direct process.env.XDG_CONFIG_HOME access exists outside the dedicated configuration module
- All configuration consumers receive configuration through dependency injection or parameters
- Configuration module has >90% test coverage with mocked environment variable scenarios
- All XDG Base Directory variables are accessed through centralized configuration accessors
- Configuration validation is implemented with clear error messages

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory and must be checked before accepting configuration-related code changes.
</enforcement>
# Isolate Environment Variable Access for Configuration Discovery: Modules That Require

These rules are ALWAYS ACTIVE for all runtime modules that access configuration values derived from environment variables, particularly those handling XDG Base Directory specification variables and configuration path resolution.

### Rules

- **R-ENV-001** MUST: Modules that require configuration values MUST receive them through dependency injection or explicit parameters rather than direct process.env access.
- **R-ENV-002** MUST: All runtime configuration derived from environment variables MUST be centralized in a dedicated configuration module (src/config or equivalent).
- **R-ENV-003** MUST: Configuration consumers MUST NOT access process.env directly outside the designated configuration module.
- **R-ENV-004** SHOULD: Configuration module SHOULD export typed configuration accessors (e.g., getXdgConfigHome()) with validation and default value handling.
- **R-ENV-005** SHOULD: Configuration module SHOULD implement validation with clear error messages for missing or invalid environment variables.

### Verify

```bash
# Detect direct process.env.XDG_CONFIG_HOME access outside config module
grep -r 'process\.env\.XDG_CONFIG_HOME' src/ --exclude-dir=config --exclude-dir=__tests__

# Detect all direct process.env access outside config module (excluding NODE_ENV and system vars)
grep -r 'process\.env\.' src/ | grep -v 'src/config/' | grep -v '__tests__' | grep -v 'NODE_ENV' | grep -v 'NODE_PATH'

# Verify configuration module test coverage
npm test -- --testPathPattern=config --coverage

# Verify all configuration consumers receive config through injection
grep -r 'getXdgConfigHome\|configService\|config:' src/ --include='*.ts' | grep -v 'src/config/'
```

**Accept when:**
- No direct process.env.XDG_CONFIG_HOME access exists outside the dedicated configuration module
- All configuration consumers receive configuration through dependency injection or function parameters
- Configuration module has >90% test coverage with mocked environment variable scenarios
- All XDG Base Directory variables are accessed through centralized configuration accessors
- Code review checklist confirms configuration injection verification for all modified modules

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep-based checks MUST pass before accepting changes. Configuration module test coverage MUST meet the >90% threshold. Direct process.env access outside the configuration module is a violation that blocks merge.
</enforcement>
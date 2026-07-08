# Use process.env for XDG_CONFIG_HOME Resolution in Configuration Loaders: Public Configuration Loading

These rules are ALWAYS ACTIVE for all configuration loading operations that resolve user-specific configuration paths in src/core/UnifiedConfigLoader.ts and src/core/ConfigLoader.ts.

### Rules

- **R-CFG-001** MUST: Public configuration loading contracts (loadUnifiedConfig, loadConfig) MUST maintain consistent XDG_CONFIG_HOME resolution behavior by accessing process.env.XDG_CONFIG_HOME directly in configuration path resolution functions.
- **R-CFG-002** MUST: Configuration loaders MUST NOT hardcode XDG_CONFIG_HOME values or user-specific configuration directory paths in source code.
- **R-CFG-003** MUST: Configuration path resolution MUST implement fallback logic when XDG_CONFIG_HOME is undefined, typically defaulting to $HOME/.config per XDG specification.
- **R-CFG-004** SHOULD: Resolved configuration directory paths SHOULD be logged at debug level to aid troubleshooting in production environments.
- **R-CFG-005** SHOULD: Resolved paths SHOULD be validated before attempting file system operations to fail fast on misconfiguration.
- **R-CFG-006** SHOULD: XDG_CONFIG_HOME dependency SHOULD be documented in public API documentation for loadUnifiedConfig and loadConfig functions.

### Verify

```bash
# Verify process.env.XDG_CONFIG_HOME is accessed in both configuration loaders
grep -r 'process\.env\.XDG_CONFIG_HOME' src/core/UnifiedConfigLoader.ts src/core/ConfigLoader.ts

# Verify no hardcoded XDG_CONFIG_HOME values exist in configuration loader files
grep -r 'XDG_CONFIG_HOME' src/core/*.ts | grep -v 'process.env' | wc -l | grep '^0$'

# Verify configuration loading tests pass with mocked XDG_CONFIG_HOME environment variables
npm test -- --grep 'configuration.*XDG'
```

**Accept when:**
- grep confirms process.env.XDG_CONFIG_HOME is accessed in both UnifiedConfigLoader.ts and ConfigLoader.ts
- No hardcoded XDG_CONFIG_HOME values exist in configuration loader files (grep returns 0 matches)
- Configuration loading tests pass with mocked XDG_CONFIG_HOME environment variables

<enforcement>
Clause Code MUST NOT skip or defer verification. All three verification commands must pass before accepting changes to configuration loading code.
</enforcement>
# Use process.env for XDG_CONFIG_HOME Resolution in Configuration Loaders: Configuration Loaders Not

These rules are ALWAYS ACTIVE for all configuration loading operations that resolve user-specific configuration paths in src/core/UnifiedConfigLoader.ts and src/core/ConfigLoader.ts.

### Rules

- **R-CFG-001** MUST_NOT: Configuration loaders MUST NOT cache process.env.XDG_CONFIG_HOME values across multiple configuration load operations unless explicitly designed for immutable environments.

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
Clause R-CFG-001 MUST be verified before any configuration loader changes are merged. Code review MUST block merge if configuration loading bypasses process.env.XDG_CONFIG_HOME. CI pipeline MUST fail if hardcoded configuration paths are detected in configuration loader modules.
</enforcement>
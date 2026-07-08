# Use process.env for XDG_CONFIG_HOME Resolution in Configuration Loaders: Configuration Loaders Provide

These rules are ALWAYS ACTIVE for all configuration loading operations that resolve user-specific configuration paths in src/core/UnifiedConfigLoader.ts and src/core/ConfigLoader.ts.

### Rules

- **R-CONFIG-001** SHOULD: Configuration loaders SHOULD provide fallback behavior when XDG_CONFIG_HOME is not set in the environment.
- **R-CONFIG-002** MUST: Access process.env.XDG_CONFIG_HOME directly in configuration path resolution functions, following the pattern established in UnifiedConfigLoader.ts and ConfigLoader.ts.
- **R-CONFIG-003** MUST: Implement fallback logic when XDG_CONFIG_HOME is undefined, typically defaulting to $HOME/.config per XDG specification.
- **R-CONFIG-004** MUST: Log the resolved configuration directory path at debug level to aid troubleshooting in production environments.
- **R-CONFIG-005** MUST: Validate resolved paths before attempting file system operations to fail fast on misconfiguration.
- **R-CONFIG-006** MUST: Document the XDG_CONFIG_HOME dependency in public API documentation for loadUnifiedConfig and loadConfig functions.
- **R-CONFIG-007** MUST NOT: Hardcode configuration paths in configuration loader files (src/core/UnifiedConfigLoader.ts, src/core/ConfigLoader.ts).

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
- Resolved configuration directory paths are logged at debug level
- Path validation occurs before file system operations
- Public API documentation includes XDG_CONFIG_HOME dependency notes

<enforcement>
Clause Code MUST NOT skip or defer verification. All rules in this file are mandatory for configuration loading code paths. Violations detected by CI pipeline grep patterns or code review must block merge. Exception requests require architecture review and documented approval with ADR reference.
</enforcement>
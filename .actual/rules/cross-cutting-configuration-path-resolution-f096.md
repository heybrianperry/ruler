# Standardize XDG_CONFIG_HOME for User-Level Configuration Discovery: Configuration Path Resolution

These rules are ALWAYS ACTIVE for all files matching the configured scope: user-level configuration file discovery in src/core/FileSystemUtils.ts, CLI handler configuration resolution in src/cli/handlers.ts, global configuration directory lookups that span user environments, and runtime environment variable access for configuration paths.

### Rules

- **R-CONFIG-001** SHOULD: Configuration path resolution SHOULD be centralized in core utilities rather than duplicated across CLI handlers and application logic.

### Verify

```bash
# Count direct XDG_CONFIG_HOME accesses in source
grep -r 'process\.env\.XDG_CONFIG_HOME' src/ | wc -l

# Verify fallback behavior is documented
grep -r 'XDG_CONFIG_HOME' src/ --include='*.ts' -A 3 | grep -E '(fallback|default|undefined)'

# Run configuration path tests
npm test -- --grep 'XDG_CONFIG_HOME'
```

**Accept when:**
- All modules that require user-level configuration paths access process.env.XDG_CONFIG_HOME with documented fallback behavior
- Error logging for configuration directory access includes the resolved path and error details
- Tests verify configuration discovery behavior with XDG_CONFIG_HOME set, unset, and pointing to invalid paths
- Configuration path resolution is centralized in a dedicated utility module, not scattered across handlers and core logic
- Resolved paths are validated for symbolic links and permissions before use

<enforcement>
Clause MUST NOT skip or defer verification. Static analysis MUST detect direct process.env access outside approved configuration modules. Code review MUST enforce XDG_CONFIG_HOME usage for user-level configuration paths. CI pipeline MUST fail if new code accesses hardcoded configuration paths instead of XDG_CONFIG_HOME.
</enforcement>
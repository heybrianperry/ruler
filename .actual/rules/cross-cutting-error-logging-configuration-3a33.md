# Standardize XDG_CONFIG_HOME for User-Level Configuration Discovery: Error Logging Configuration

These rules are ALWAYS ACTIVE for all files in `src/core/FileSystemUtils.ts` and `src/cli/handlers.ts` that perform user-level configuration directory discovery and access.

### Rules

- **R-XDG-001** MUST: Error logging for configuration directory access failures MUST include the resolved path and error context.

### Verify

```bash
# Check for XDG_CONFIG_HOME access patterns
grep -r 'process\.env\.XDG_CONFIG_HOME' src/ | wc -l

# Verify fallback behavior is documented
grep -r 'XDG_CONFIG_HOME' src/ --include='*.ts' -A 3 | grep -E '(fallback|default|undefined)'

# Run configuration discovery tests
npm test -- --grep 'XDG_CONFIG_HOME'
```

**Accept when:**
- All modules that require user-level configuration paths access `process.env.XDG_CONFIG_HOME` with documented fallback behavior
- Error logging for configuration directory access includes the resolved path and error details
- Tests verify configuration discovery behavior with `XDG_CONFIG_HOME` set, unset, and pointing to invalid paths
- No hardcoded configuration paths exist outside approved fallback scenarios

<enforcement>
Claude Code MUST NOT skip or defer verification. All error logging for configuration directory access failures MUST include resolved path and error context before code is considered compliant.
</enforcement>
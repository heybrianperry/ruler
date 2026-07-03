# Standardize XDG_CONFIG_HOME for User-Level Configuration Discovery: Security Sensitive Configuration

These rules are ALWAYS ACTIVE for all files matching user-level configuration discovery logic, including src/core/FileSystemUtils.ts, src/cli/handlers.ts, and any modules that resolve global configuration paths through environment variables.

### Rules

- **R-XDG-001** SHOULD: Security-sensitive configuration paths SHOULD validate that resolved directories are not symbolic links when appropriate.

### Verify

```bash
# Count direct XDG_CONFIG_HOME access patterns
grep -r 'process\.env\.XDG_CONFIG_HOME' src/ | wc -l

# Verify fallback behavior is documented
grep -r 'XDG_CONFIG_HOME' src/ --include='*.ts' -A 3 | grep -E '(fallback|default|undefined)'

# Run configuration discovery tests
npm test -- --grep 'XDG_CONFIG_HOME'
```

**Accept when:**
- All modules that require user-level configuration paths access process.env.XDG_CONFIG_HOME with documented fallback behavior
- Error logging for configuration directory access includes the resolved path and error details
- Tests verify configuration discovery behavior with XDG_CONFIG_HOME set, unset, and pointing to invalid paths
- Symbolic link validation is performed on resolved configuration directories before use

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis, code review, and integration tests are mandatory before accepting configuration path changes.
</enforcement>
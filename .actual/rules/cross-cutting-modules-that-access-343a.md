# Standardize XDG_CONFIG_HOME for User-Level Configuration Discovery: Modules That Access

These rules are ALWAYS ACTIVE for all modules that access user-level configuration directories, including src/core/FileSystemUtils.ts, src/cli/handlers.ts, and any code performing global configuration directory lookups that span user environments.

### Rules

- **R-XDG-001** MUST: Modules that access XDG_CONFIG_HOME MUST handle undefined values with appropriate fallback logic (e.g., path.join(os.homedir(), '.config') on POSIX systems).
- **R-XDG-002** MUST: Configuration directory access MUST include validation for symbolic links and verification that the containing directory is within expected root boundaries.
- **R-XDG-003** MUST: Error logging for configuration directory access MUST include the resolved path and full error details.
- **R-XDG-004** SHOULD: XDG_CONFIG_HOME access SHOULD be centralized in a dedicated configuration utility module to reduce duplication across FileSystemUtils.ts and handlers.ts.
- **R-XDG-005** SHOULD: User-facing documentation (README and CLI help text) SHOULD document the XDG_CONFIG_HOME requirement with examples for setting the variable in different shells.

### Verify

```bash
# Count direct process.env.XDG_CONFIG_HOME accesses
grep -r 'process\.env\.XDG_CONFIG_HOME' src/ | wc -l

# Verify fallback behavior is documented
grep -r 'XDG_CONFIG_HOME' src/ --include='*.ts' -A 3 | grep -E '(fallback|default|undefined)'

# Run XDG_CONFIG_HOME-specific tests
npm test -- --grep 'XDG_CONFIG_HOME'
```

**Accept when:**
- All modules that require user-level configuration paths access process.env.XDG_CONFIG_HOME with documented fallback behavior
- Error logging for configuration directory access includes the resolved path and error details
- Tests verify configuration discovery behavior with XDG_CONFIG_HOME set, unset, and pointing to invalid paths
- Configuration path logic is centralized in approved configuration modules, not scattered across the codebase
- No hardcoded configuration paths exist outside fallback scenarios

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis MUST detect direct process.env access outside approved configuration modules. Code review MUST require XDG_CONFIG_HOME usage for user-level configuration paths. Integration tests MUST validate configuration discovery across different environment variable states. CI pipeline MUST fail if new code accesses hardcoded configuration paths instead of XDG_CONFIG_HOME.
</enforcement>
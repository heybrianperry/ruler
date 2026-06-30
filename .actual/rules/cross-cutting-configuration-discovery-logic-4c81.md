# Standardize XDG_CONFIG_HOME for User-Level Configuration Discovery: Configuration Discovery Logic

These rules are ALWAYS ACTIVE for all files matching the configured scope: user-level configuration file discovery in src/core/FileSystemUtils.ts, CLI handler configuration resolution in src/cli/handlers.ts, global configuration directory lookups that span user environments, and runtime environment variable access for configuration paths.

### Rules

- **R-CONFIG-001** MUST: Configuration discovery logic MUST read process.env.XDG_CONFIG_HOME as the primary source for user-level configuration directory paths.
- **R-CONFIG-002** MUST: All modules that require user-level configuration paths MUST access process.env.XDG_CONFIG_HOME with documented fallback behavior (e.g., path.join(os.homedir(), '.config') on POSIX systems when undefined).
- **R-CONFIG-003** MUST: Error logging for configuration directory access MUST include the resolved path and error details.
- **R-CONFIG-004** MUST: Resolved configuration directories MUST be validated for symbolic links and permissions before reading configuration files.
- **R-CONFIG-005** SHOULD: Configuration discovery logic SHOULD be centralized in a dedicated configuration utility module to reduce duplication across FileSystemUtils.ts and handlers.ts.
- **R-CONFIG-006** SHOULD: Configuration path resolution SHOULD be documented in README and CLI help text, including examples for setting XDG_CONFIG_HOME in different shells.

### Verify

```bash
# Count direct XDG_CONFIG_HOME accesses
grep -r 'process\.env\.XDG_CONFIG_HOME' src/ | wc -l

# Check for fallback/default/undefined handling
grep -r 'XDG_CONFIG_HOME' src/ --include='*.ts' -A 3 | grep -E '(fallback|default|undefined)'

# Run configuration discovery tests
npm test -- --grep 'XDG_CONFIG_HOME'

# Verify no hardcoded configuration paths outside approved modules
grep -r "path\.join.*\.config" src/ --include='*.ts' | grep -v 'FileSystemUtils\|handlers' | wc -l
```

**Accept when:**
- All modules that require user-level configuration paths access process.env.XDG_CONFIG_HOME with documented fallback behavior
- Error logging for configuration directory access includes the resolved path and error details
- Tests verify configuration discovery behavior with XDG_CONFIG_HOME set, unset, and pointing to invalid paths
- Configuration discovery logic is centralized in a dedicated module or clearly documented in each access point
- No hardcoded configuration paths exist outside of fallback scenarios

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis MUST detect direct process.env access outside approved configuration modules. Code review MUST enforce XDG_CONFIG_HOME usage for user-level configuration paths. Integration tests MUST validate configuration discovery across different environment variable states. CI pipeline MUST fail if new code accesses hardcoded configuration paths instead of XDG_CONFIG_HOME.
</enforcement>
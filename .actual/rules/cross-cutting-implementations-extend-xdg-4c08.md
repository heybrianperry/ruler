# Standardize XDG_CONFIG_HOME for User-Level Configuration Discovery: Implementations Extend Xdg

These rules are ALWAYS ACTIVE for all files matching the configured scope: user-level configuration file discovery in src/core/FileSystemUtils.ts, CLI handler configuration resolution in src/cli/handlers.ts, global configuration directory lookups that span user environments, and runtime environment variable access for configuration paths.

### Rules

- **R-XDG-001** MAY: Implementations MAY extend XDG_CONFIG_HOME-based discovery with additional platform-specific fallbacks for compatibility.
- **R-XDG-002** MUST: All modules that require user-level configuration paths access process.env.XDG_CONFIG_HOME with documented fallback behavior.
- **R-XDG-003** MUST: Centralize XDG_CONFIG_HOME access in a dedicated configuration utility module to reduce duplication between FileSystemUtils.ts and handlers.ts.
- **R-XDG-004** MUST: Implement fallback logic: if XDG_CONFIG_HOME is undefined, default to path.join(os.homedir(), '.config') on POSIX systems.
- **R-XDG-005** MUST: Add validation for resolved configuration directories: check for symbolic links, verify containing directory is within expected root, and log errors with full context.
- **R-XDG-006** MUST: Document the XDG_CONFIG_HOME requirement in README and CLI help text, including examples for setting the variable in different shells.
- **R-XDG-007** MUST: Error logging for configuration directory access includes the resolved path and error details.
- **R-XDG-008** MUST NOT: Access hardcoded configuration paths instead of XDG_CONFIG_HOME in new code.
- **R-XDG-009** SHOULD: Validate resolved paths for symbolic links and permissions before reading configuration files.

### Verify

```bash
# Count direct XDG_CONFIG_HOME access points
grep -r 'process\.env\.XDG_CONFIG_HOME' src/ | wc -l

# Check for fallback and default handling
grep -r 'XDG_CONFIG_HOME' src/ --include='*.ts' -A 3 | grep -E '(fallback|default|undefined)'

# Run XDG_CONFIG_HOME-specific tests
npm test -- --grep 'XDG_CONFIG_HOME'

# Verify no hardcoded configuration paths in new modules
grep -r '\.config' src/ --include='*.ts' | grep -v 'XDG_CONFIG_HOME' | grep -v 'test' | grep -v '.config/'
```

**Accept when:**
- All modules that require user-level configuration paths access process.env.XDG_CONFIG_HOME with documented fallback behavior
- Error logging for configuration directory access includes the resolved path and error details
- Tests verify configuration discovery behavior with XDG_CONFIG_HOME set, unset, and pointing to invalid paths
- Configuration path logic is centralized in a dedicated utility module, not scattered across FileSystemUtils.ts and handlers.ts
- Fallback logic defaults to path.join(os.homedir(), '.config') when XDG_CONFIG_HOME is undefined
- Resolved configuration directories are validated for symbolic links and permissions before use

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis MUST detect direct process.env access outside approved configuration modules. Code review MUST verify XDG_CONFIG_HOME usage for user-level configuration paths. Integration tests MUST validate configuration discovery across different environment variable states. CI pipeline MUST fail if new code accesses hardcoded configuration paths instead of XDG_CONFIG_HOME.
</enforcement>
# Use XDG_CONFIG_HOME Environment Variable for Global Configuration Discovery: Configuration Discovery Logic

These rules are ALWAYS ACTIVE for all modules that discover or resolve global configuration directories, including src/core/FileSystemUtils.ts, src/cli/handlers.ts, and any code that accesses process.env.XDG_CONFIG_HOME for configuration file location.

### Rules

- **R-XDG-001** MUST NOT: Configuration discovery logic MUST NOT hardcode absolute paths to global configuration directories.
- **R-XDG-002** MUST: All modules requiring global configuration discovery MUST read XDG_CONFIG_HOME from process.env.
- **R-XDG-003** MUST: Error handling MUST be present for all configuration directory access operations with descriptive logging using console.error.
- **R-XDG-004** MUST: Security validation utilities (assertNotSymbolicLink, assertContainingDirectoryInsideRoot) MUST be applied to paths derived from environment variables.
- **R-XDG-005** SHOULD: Implement fallback logic that checks XDG_CONFIG_HOME first, then falls back to platform-specific defaults like ~/.config on Unix systems.
- **R-XDG-006** SHOULD: Use path resolution utilities (path.resolve, path.join) to normalize paths obtained from environment variables before filesystem operations.
- **R-XDG-007** SHOULD: Import configuration discovery utilities from src/core/FileSystemUtils.ts to ensure consistent behavior across modules.

### Verify

```bash
# Check for XDG_CONFIG_HOME environment variable access patterns
grep -r 'process\.env\.XDG_CONFIG_HOME' src/ --include='*.ts'

# Check for error handling around global config discovery
grep -r 'console\.error.*global.*config' src/ --include='*.ts'

# Verify security validation utilities are used
grep -r 'assertNotSymbolicLink\|assertContainingDirectoryInsideRoot' src/core/FileSystemUtils.ts

# Ensure no hardcoded configuration paths in discovery logic
grep -r '/etc/\|~/.config' src/core/FileSystemUtils.ts src/cli/handlers.ts | grep -v 'fallback\|default' || echo 'No hardcoded paths found'
```

**Accept when:**
- All modules requiring global configuration discovery read XDG_CONFIG_HOME from process.env
- Error handling is present for all configuration directory access operations with descriptive logging
- Security validation utilities are applied to paths derived from environment variables
- No hardcoded absolute paths to global configuration directories exist in configuration discovery logic
- Fallback logic is implemented for cases where XDG_CONFIG_HOME is unset or invalid
- Path normalization utilities are used before filesystem operations

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All configuration discovery implementations MUST be reviewed against R-XDG-001 through R-XDG-007 before acceptance.
</enforcement>
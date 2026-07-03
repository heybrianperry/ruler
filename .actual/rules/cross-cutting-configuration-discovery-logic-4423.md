# Use XDG_CONFIG_HOME Environment Variable for Global Configuration Discovery: Configuration Discovery Logic

These rules are ALWAYS ACTIVE for all modules that discover or resolve global configuration directories, including src/core/FileSystemUtils.ts, src/cli/handlers.ts, and any code that needs to locate user or system-wide configuration files.

### Rules

- **R-XDG-001** MUST: Configuration discovery logic MUST read the XDG_CONFIG_HOME environment variable from process.env when determining global configuration directory paths.
- **R-XDG-002** MUST: All XDG_CONFIG_HOME access MUST be wrapped with try-catch blocks and errors MUST be logged using console.error with descriptive context.
- **R-XDG-003** MUST: Paths derived from XDG_CONFIG_HOME MUST be validated using security utilities (e.g., assertContainingDirectoryInsideRoot, assertNotSymbolicLink) before filesystem operations.
- **R-XDG-004** SHOULD: Configuration discovery SHOULD implement fallback logic that checks XDG_CONFIG_HOME first, then falls back to platform-specific defaults like ~/.config on Unix systems.
- **R-XDG-005** SHOULD: Path resolution utilities (path.resolve, path.join) SHOULD be used to normalize paths obtained from environment variables before filesystem operations.
- **R-XDG-006** SHOULD: Configuration discovery utilities SHOULD be imported from src/core/FileSystemUtils.ts to ensure consistent behavior across modules.

### Verify

```bash
# Check for XDG_CONFIG_HOME environment variable access patterns
grep -r 'process\.env\.XDG_CONFIG_HOME' src/ --include='*.ts'

# Check for error handling around global config access
grep -r 'console\.error.*global.*config' src/ --include='*.ts'

# Verify security validation utilities are applied
grep -r 'assertNotSymbolicLink\|assertContainingDirectoryInsideRoot' src/core/FileSystemUtils.ts
```

**Accept when:**
- All modules requiring global configuration discovery read XDG_CONFIG_HOME from process.env
- Error handling is present for all configuration directory access operations with descriptive logging
- Security validation utilities are applied to paths derived from environment variables
- Fallback logic to platform-specific defaults is implemented where appropriate
- Path normalization using path.resolve or path.join is applied to environment variable values

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code affecting global configuration discovery.
</enforcement>
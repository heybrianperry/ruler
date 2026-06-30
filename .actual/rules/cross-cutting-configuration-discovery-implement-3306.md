# Use XDG_CONFIG_HOME Environment Variable for Global Configuration Discovery: Configuration Discovery Implement

These rules are ALWAYS ACTIVE for all modules that discover or access global configuration directories, including src/core/FileSystemUtils.ts, src/cli/handlers.ts, and any code that needs to locate user or system-wide configuration files.

### Rules

- **R-CONFIG-001** MUST: Configuration discovery MUST implement error handling for cases where the global config directory is inaccessible or does not exist.
- **R-CONFIG-002** MUST: All modules requiring global configuration discovery MUST read XDG_CONFIG_HOME from process.env.
- **R-CONFIG-003** MUST: XDG_CONFIG_HOME access MUST be wrapped with try-catch blocks and logged using console.error with descriptive context.
- **R-CONFIG-004** MUST: Security validation utilities (assertNotSymbolicLink, assertContainingDirectoryInsideRoot) MUST be applied to paths derived from environment variables.
- **R-CONFIG-005** SHOULD: Implement fallback logic that checks XDG_CONFIG_HOME first, then falls back to platform-specific defaults like ~/.config on Unix systems.
- **R-CONFIG-006** SHOULD: Use path resolution utilities (path.resolve, path.join) to normalize paths obtained from environment variables before filesystem operations.

### Verify

```bash
# Check for XDG_CONFIG_HOME environment variable access patterns
grep -r 'process\.env\.XDG_CONFIG_HOME' src/ --include='*.ts'

# Check for error handling in global config operations
grep -r 'console\.error.*global.*config' src/ --include='*.ts'

# Verify security validation utilities are used
grep -r 'assertNotSymbolicLink\|assertContainingDirectoryInsideRoot' src/core/FileSystemUtils.ts
```

**Accept when:**
- All modules requiring global configuration discovery read XDG_CONFIG_HOME from process.env
- Error handling is present for all configuration directory access operations with descriptive logging
- Security validation utilities are applied to paths derived from environment variables
- Fallback logic exists for cases where XDG_CONFIG_HOME is unset or invalid
- Path normalization is performed using path.resolve or path.join before filesystem operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration discovery code MUST satisfy R-CONFIG-001 through R-CONFIG-006 before acceptance. Pull requests introducing hardcoded configuration paths or direct filesystem access without environment variable checks MUST be rejected.
</enforcement>
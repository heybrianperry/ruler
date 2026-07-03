# Use XDG_CONFIG_HOME Environment Variable for Global Configuration Discovery: Error Messages Related

These rules are ALWAYS ACTIVE for all files that access global configuration directories, read from process.env.XDG_CONFIG_HOME, or handle configuration discovery errors across the codebase.

### Rules

- **R-XDG-001** SHOULD: Error messages related to configuration directory access SHOULD be logged using console.error with descriptive context including the attempted directory path.

### Verify

```bash
# Check for XDG_CONFIG_HOME environment variable access patterns
grep -r 'process\.env\.XDG_CONFIG_HOME' src/ --include='*.ts'

# Verify error logging for global config operations
grep -r 'console\.error.*global.*config' src/ --include='*.ts'

# Confirm security validation utilities are applied to environment-derived paths
grep -r 'assertNotSymbolicLink\|assertContainingDirectoryInsideRoot' src/core/FileSystemUtils.ts
```

**Accept when:**
- All modules requiring global configuration discovery read XDG_CONFIG_HOME from process.env
- Error handling is present for all configuration directory access operations with descriptive logging using console.error
- Security validation utilities are applied to paths derived from environment variables
- Configuration discovery errors include the attempted directory path in the error message

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All configuration directory access must include proper error logging with descriptive context.
</enforcement>
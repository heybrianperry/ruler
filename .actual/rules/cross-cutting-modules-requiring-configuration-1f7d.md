# Use XDG_CONFIG_HOME Environment Variable for Global Configuration Discovery: Modules Requiring Configuration

These rules are ALWAYS ACTIVE for all modules requiring global configuration discovery, particularly those in src/core/FileSystemUtils.ts, src/cli/handlers.ts, and any other module that needs to locate user or system-wide configuration files.

### Rules

- **R-XDG-001** SHOULD: Modules requiring configuration discovery SHOULD import and use shared utilities from core modules (e.g., FileSystemUtils) to ensure consistent behavior across the codebase.
- **R-XDG-002** MUST: All modules accessing global configuration directories MUST wrap XDG_CONFIG_HOME access with try-catch blocks and log errors using console.error with descriptive context.
- **R-XDG-003** MUST: Configuration discovery MUST implement fallback logic that checks XDG_CONFIG_HOME first, then falls back to platform-specific defaults like ~/.config on Unix systems.
- **R-XDG-004** MUST: Paths derived from environment variables MUST be validated using security utilities (e.g., assertContainingDirectoryInsideRoot, assertNotSymbolicLink) before filesystem operations.
- **R-XDG-005** MUST: Path resolution utilities (path.resolve, path.join) MUST be used to normalize paths obtained from environment variables before any filesystem operations.

### Verify

```bash
# Check for XDG_CONFIG_HOME usage patterns
grep -r 'process\.env\.XDG_CONFIG_HOME' src/ --include='*.ts'

# Verify error handling for global config access
grep -r 'console\.error.*global.*config' src/ --include='*.ts'

# Confirm security validation utilities are applied
grep -r 'assertNotSymbolicLink\|assertContainingDirectoryInsideRoot' src/core/FileSystemUtils.ts

# Check for hardcoded configuration paths (should be minimal/absent)
grep -r '"/etc/\|~/.config' src/ --include='*.ts' | grep -v 'fallback\|default'
```

**Accept when:**
- All modules requiring global configuration discovery read XDG_CONFIG_HOME from process.env
- Error handling is present for all configuration directory access operations with descriptive logging
- Security validation utilities are applied to paths derived from environment variables
- Path normalization using path.resolve or path.join is performed on environment variable values
- Fallback logic to platform-specific defaults is implemented and tested

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review acceptance.
</enforcement>
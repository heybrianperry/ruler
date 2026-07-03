# Standardize Console-Based Logging for Public API Boundaries: Error Messages Include

These rules are ALWAYS ACTIVE for all TypeScript modules in src/ that export public interfaces, types, or functions; CLI handler modules that interact with user input or command-line arguments; core utility modules that perform file system operations, configuration loading, or environment variable access; and agent implementations that coordinate external resources or maintain state.

### Rules

- **R-CONSOLE-001** SHOULD: Error messages SHOULD include actionable context such as file paths, error messages from underlying operations, or configuration keys to facilitate debugging.

### Verify

```bash
# Verify prefixed console.error and console.warn statements
grep -r 'console\.error\|console\.warn' src/ --include='*.ts' | grep -E '\[(ruler|ERROR_PREFIX|WARN_PREFIX)' || echo 'No prefixed logging found'

# Verify no console.log in production code
grep -r 'console\.log' src/ --include='*.ts' --exclude='*.test.ts' | grep -v 'test\|spec' && echo 'Found console.log in non-test files' || echo 'No console.log in production code'

# Verify logging tests exist
npm test -- --grep 'logging' 2>&1 | grep -E 'pass|fail' || echo 'No logging tests found'
```

**Accept when:**
- All console.error and console.warn statements in src/ modules with public exports include contextual prefixes identifying the subsystem
- No console.log statements exist in production code paths (excluding test files and development utilities)
- Logging statements at configuration loading and file system boundaries include actionable context (paths, error messages, keys)

<enforcement>
Claude Code MUST NOT skip or defer verification of console logging patterns in public API modules. All violations MUST be flagged during code review and CI pipeline checks.
</enforcement>
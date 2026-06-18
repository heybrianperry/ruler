# Standardize Console-Based Logging for Public API Boundaries: Logging Statements Not

These rules are ALWAYS ACTIVE for all TypeScript modules in src/ that export public interfaces, types, or functions; CLI handler modules that interact with user input or command-line arguments; core utility modules that perform file system operations, configuration loading, or environment variable access; and agent implementations that coordinate external resources or maintain state.

### Rules

- **R-LOGGING-001** MUST_NOT: Logging statements MUST NOT be used for application-level output intended for end-user consumption or programmatic parsing.
- **R-LOGGING-002** MUST: All console.error and console.warn statements in public API modules MUST include contextual prefixes identifying the subsystem (e.g., [ruler], ERROR_PREFIX, WARN_PREFIX).
- **R-LOGGING-003** MUST: Reserve console.error for error-level diagnostics and console.warn for warning-level diagnostics; avoid console.log for operational messages to maintain clean stdout.
- **R-LOGGING-004** SHOULD: Use template literals to include contextual information (file paths, error messages, configuration keys) in logging statements for actionable diagnostics.
- **R-LOGGING-005** SHOULD: When implementing verbose logging modes, use conditional checks (e.g., if (verbose)) before console calls to avoid performance overhead in production.

### Verify

```bash
# Verify prefixed logging in public API modules
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
- Verbose logging uses conditional checks to avoid performance overhead

<enforcement>
Claude Code MUST NOT skip or defer verification of logging patterns in public API modules. All console-based logging must include contextual prefixes and follow the prescribed levels (error/warn only, no log).
</enforcement>
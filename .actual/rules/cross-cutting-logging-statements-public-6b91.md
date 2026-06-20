# Standardize Console-Based Logging for Public API Boundaries: Logging Statements Public

These rules are ALWAYS ACTIVE for all TypeScript modules in src/ that export public interfaces, types, or functions; CLI handler modules that interact with user input or command-line arguments; core utility modules that perform file system operations, configuration loading, or environment variable access; and agent implementations that coordinate external resources or maintain state.

### Rules

- **R-LOG-001** MUST: Logging statements at public API boundaries MUST include contextual prefixes that identify the subsystem or operation (e.g., [ruler], [ruler:verbose], ERROR_PREFIX).
- **R-LOG-002** MUST: Use console.error for error-level diagnostics and console.warn for warning-level diagnostics; avoid console.log for operational messages to maintain clean stdout.
- **R-LOG-003** MUST: Define shared constants for logging prefixes (e.g., ERROR_PREFIX, WARN_PREFIX, VERBOSE_PREFIX) in src/constants.ts to ensure consistency across modules.
- **R-LOG-004** SHOULD: Use template literals to include contextual information (file paths, error messages, configuration keys) in logging statements for actionable diagnostics.
- **R-LOG-005** SHOULD: When implementing verbose logging modes, use conditional checks (e.g., if (verbose)) before console calls to avoid performance overhead in production.

### Verify

```bash
# Verify all console.error and console.warn statements include contextual prefixes
grep -r 'console\.error\|console\.warn' src/ --include='*.ts' | grep -E '\[(ruler|ERROR_PREFIX|WARN_PREFIX)' || echo 'No prefixed logging found'

# Verify no console.log statements in production code
grep -r 'console\.log' src/ --include='*.ts' --exclude='*.test.ts' | grep -v 'test\|spec' && echo 'Found console.log in non-test files' || echo 'No console.log in production code'

# Verify logging tests exist
npm test -- --grep 'logging' 2>&1 | grep -E 'pass|fail' || echo 'No logging tests found'
```

**Accept when:**
- All console.error and console.warn statements in src/ modules with public exports include contextual prefixes identifying the subsystem
- No console.log statements exist in production code paths (excluding test files and development utilities)
- Logging statements at configuration loading and file system boundaries include actionable context (paths, error messages, keys)
- Shared logging prefix constants are defined in src/constants.ts and used consistently across modules

<enforcement>
Claude Code MUST NOT skip or defer verification of these logging rules. All public API boundary modules MUST comply with prefixed console logging before code review approval.
</enforcement>
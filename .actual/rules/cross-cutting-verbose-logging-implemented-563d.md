# Standardize Console-Based Logging for Public API Boundaries: Verbose Logging Implemented

These rules are ALWAYS ACTIVE for all TypeScript modules in src/ that export public interfaces, types, or functions; CLI handler modules that interact with user input or command-line arguments; core utility modules that perform file system operations, configuration loading, or environment variable access; and agent implementations that coordinate external resources or maintain state.

### Rules

- **R-LOG-001** MAY: Verbose logging MAY be implemented using conditional console methods with distinguishable prefixes to support diagnostic modes without polluting standard output.
- **R-LOG-002** MUST: All console.error and console.warn statements in modules with public exports include contextual prefixes identifying the subsystem (e.g., [ruler], ERROR_PREFIX, WARN_PREFIX).
- **R-LOG-003** MUST: Reserve console.error for error-level diagnostics and console.warn for warning-level diagnostics; avoid console.log for operational messages to maintain clean stdout.
- **R-LOG-004** MUST: No console.log statements exist in production code paths (excluding test files and development utilities).
- **R-LOG-005** SHOULD: Use template literals to include contextual information (file paths, error messages, configuration keys) in logging statements for actionable diagnostics.
- **R-LOG-006** SHOULD: Define shared constants for logging prefixes (e.g., ERROR_PREFIX, WARN_PREFIX, VERBOSE_PREFIX) in src/constants.ts to ensure consistency across modules.
- **R-LOG-007** SHOULD: When implementing verbose logging modes, use conditional checks (e.g., if (verbose)) before console calls to avoid performance overhead in production.

### Verify

```bash
# Verify prefixed console.error and console.warn usage
grep -r 'console\.error\|console\.warn' src/ --include='*.ts' | grep -E '\[(ruler|ERROR_PREFIX|WARN_PREFIX)' || echo 'No prefixed logging found'

# Verify no console.log in production code
grep -r 'console\.log' src/ --include='*.ts' --exclude='*.test.ts' | grep -v 'test\|spec' && echo 'Found console.log in non-test files' || echo 'No console.log in production code'

# Verify logging tests exist
npm test -- --grep 'logging' 2>&1 | grep -E 'pass|fail' || echo 'No logging tests found'
```

**Accept when:**
- All console.error and console.warn statements in src/ modules with public exports include contextual prefixes identifying the subsystem.
- No console.log statements exist in production code paths (excluding test files and development utilities).
- Logging statements at configuration loading and file system boundaries include actionable context (paths, error messages, keys).

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All console-based logging in public API boundary modules MUST conform to the prefixing and level conventions defined above.
</enforcement>
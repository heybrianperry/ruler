# Standardize Console-Based Logging for Public API Boundaries: Modules Handling External

These rules are ALWAYS ACTIVE for all TypeScript modules in src/ that export public interfaces, types, or functions; CLI handler modules that interact with user input or command-line arguments; core utility modules that perform file system operations, configuration loading, or environment variable access; and agent implementations that coordinate external resources or maintain state.

### Rules

- **R-CONSOLE-001** SHOULD: Modules handling external configuration sources (environment variables, file system paths, user input) SHOULD log warnings when encountering non-fatal issues such as missing optional files or fallback behavior.
- **R-CONSOLE-002** MUST: All console.error and console.warn statements in modules with public exports MUST include contextual prefixes identifying the subsystem (e.g., [ruler], ERROR_PREFIX, WARN_PREFIX).
- **R-CONSOLE-003** MUST: console.log MUST NOT be used in production code paths; reserve console.error for error-level diagnostics and console.warn for warning-level diagnostics to maintain clean stdout.
- **R-CONSOLE-004** SHOULD: Logging statements at configuration loading and file system boundaries SHOULD include actionable context such as file paths, error messages, and configuration keys.
- **R-CONSOLE-005** MAY: Verbose logging modes MAY use conditional checks (e.g., if (verbose)) before console calls to avoid performance overhead in production.

### Verify

```bash
# Verify prefixed console logging at public API boundaries
grep -r 'console\.error\|console\.warn' src/ --include='*.ts' | grep -E '\[(ruler|ERROR_PREFIX|WARN_PREFIX)' || echo 'No prefixed logging found'

# Verify no console.log in production code
grep -r 'console\.log' src/ --include='*.ts' --exclude='*.test.ts' | grep -v 'test\|spec' && echo 'Found console.log in non-test files' || echo 'No console.log in production code'

# Verify logging test coverage
npm test -- --grep 'logging' 2>&1 | grep -E 'pass|fail' || echo 'No logging tests found'
```

**Accept when:**
- All console.error and console.warn statements in src/ modules with public exports include contextual prefixes identifying the subsystem
- No console.log statements exist in production code paths (excluding test files and development utilities)
- Logging statements at configuration loading and file system boundaries include actionable context (paths, error messages, keys)
- Verbose logging uses conditional checks to avoid performance overhead

<enforcement>
Claude Code MUST NOT skip or defer verification of console logging patterns in modules touching public API boundaries. All violations MUST be flagged during code review with requests for contextual prefix additions or log level corrections.
</enforcement>
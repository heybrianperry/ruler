# Standardize Console-Based Logging for Public API Boundaries: Modules Exposing Public

These rules are ALWAYS ACTIVE for all TypeScript modules in src/ that expose public API contracts (exported interfaces, types, or functions), CLI handler modules, core utility modules performing file system operations or configuration loading, and agent implementations coordinating external resources.

### Rules

- **R-CONSOLE-001** MUST: All modules exposing public API contracts (identified by exported interfaces, types, or functions) MUST use console.error for error-level diagnostics and console.warn for warning-level diagnostics.
- **R-CONSOLE-002** MUST: All console.error and console.warn statements in public API modules MUST include contextual prefixes (e.g., [ruler], ERROR_PREFIX, WARN_PREFIX) to distinguish operational messages from application output.
- **R-CONSOLE-003** MUST: No console.log statements SHALL exist in production code paths at public API boundaries; reserve console.log for test files and development utilities only.
- **R-CONSOLE-004** SHOULD: Logging statements at configuration loading and file system boundaries SHOULD include actionable context (file paths, error messages, configuration keys).
- **R-CONSOLE-005** SHOULD: When implementing verbose logging modes, use conditional checks (e.g., if (verbose)) before console calls to avoid performance overhead in production.

### Verify

```bash
# Verify prefixed console.error and console.warn usage in public API modules
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
- Verbose logging modes use conditional checks to avoid performance overhead

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All modules exposing public API contracts MUST comply with R-CONSOLE-001 through R-CONSOLE-005 before code review approval.
</enforcement>
# Native Console API for Error and Warning Logging: Error Warning Logging Use Native Javascript

These rules are ALWAYS ACTIVE for all error and warning logging code across agent layers, CLI handlers, and core filesystem utilities.

### Rules

- **R-CONSOLE-001** MUST: All error and warning logging MUST use the native JavaScript Console API (console.warn and console.error methods) rather than external logging libraries.
- **R-CONSOLE-002** MUST: When catching exceptions, pass the error object as a second argument to console methods to preserve stack traces for debugging.
- **R-CONSOLE-003** SHOULD: Use template literals for message formatting to include contextual information such as file paths, operation names, or component identifiers.
- **R-CONSOLE-004** SHOULD: Consider prefixing messages with component identifiers in square brackets to aid log filtering in complex workflows.
- **R-CONSOLE-005** SHOULD: Use console.warn for recoverable errors and console.error for fatal conditions to maintain consistent severity distinction.

### Verify

```bash
# Discover and inspect source files in agent, CLI, and core utility directories
find . -type f -name '*.js' -o -name '*.ts' | grep -E '(agent|cli|core|util)' | head -20

# Search for all console method calls to verify logging patterns
grep -r 'console\.(warn|error)' --include='*.js' --include='*.ts' | head -20

# Verify no external logging library imports are present
grep -r "require.*log\|import.*log" --include='*.js' --include='*.ts' --include='package.json' | grep -v 'console' | head -20

# Review error handling paths for warn/error distinction
grep -r 'console\.warn\|console\.error' --include='*.js' --include='*.ts' -B 2 -A 2 | head -40
```

**Accept when:**
- All error and warning logging across agent, CLI, and core layers uses console.warn or console.error exclusively
- No external logging library dependencies are present in dependency manifests
- Error severity distinction is consistent: console.warn for recoverable errors, console.error for fatal errors
- Error objects are passed as second arguments to console methods to preserve stack traces
- Messages include contextual information via template literals or component identifiers

<enforcement>
Claude Code MUST NOT skip or defer verification. All error logging must be audited against R-CONSOLE-001 through R-CONSOLE-005 before code review approval.
</enforcement>
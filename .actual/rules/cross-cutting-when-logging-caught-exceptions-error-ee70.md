# Native Console API for Error and Warning Logging: When Logging Caught Exceptions Error Object

These rules are ALWAYS ACTIVE for all error and warning logging code across agent layer, CLI layer, and core utility files that handle file I/O operations, user-facing error messages, and system-level errors.

### Rules

- **R-CONSOLE-001** SHOULD: When logging caught exceptions, the error object SHOULD be passed as an additional argument to console methods for stack trace preservation.
- **R-CONSOLE-002** SHOULD: Use console.warn for recoverable errors and console.error for fatal errors to maintain consistent severity distinction.
- **R-CONSOLE-003** SHOULD: Use template literals for message formatting to include contextual information such as file paths, operation names, or component identifiers.
- **R-CONSOLE-004** SHOULD: Consider prefixing messages with component identifiers in square brackets to aid log filtering in complex workflows.

### Verify

```bash
# Discover and inspect source files in agent, CLI, and core utility directories
find . -type f -name '*.js' -o -name '*.ts' | xargs grep -l 'console\.(warn|error)' | head -20

# Search the codebase for console method calls and verify no external logging library imports
grep -r 'require.*log\|import.*log' --include='*.js' --include='*.ts' | grep -v node_modules | grep -v '.actual'

# Review error handling paths to confirm warn/error distinction
grep -r 'console\.warn\|console\.error' --include='*.js' --include='*.ts' | grep -v node_modules | grep -v '.actual'

# Verify error objects are passed as arguments to console methods
grep -r 'console\.\(warn\|error\).*err' --include='*.js' --include='*.ts' | grep -v node_modules | grep -v '.actual'
```

**Accept when:**
- All error and warning logging across agent, CLI, and core layers uses console.warn or console.error exclusively
- No external logging library dependencies are present in dependency manifests
- Error severity distinction is consistent: console.warn for recoverable errors, console.error for fatal errors
- Error objects are passed as additional arguments to console methods to preserve stack traces

<enforcement>
Claude Code MUST NOT skip or defer verification. All error logging must conform to these rules before code is accepted.
</enforcement>
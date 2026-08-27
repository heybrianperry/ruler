# Native Console API for Error and Warning Logging: Recoverable Errors Non Fatal Warnings Use

These rules are ALWAYS ACTIVE for all error handling code across agent layer, CLI layer, and core utilities, including file I/O operations, user-facing error messages in CLI handlers, and system-level errors in filesystem utilities.

### Rules

- **R-CONSOLE-001** MUST: Recoverable errors and non-fatal warnings MUST use `console.warn()` to distinguish them from fatal errors.
- **R-CONSOLE-002** MUST: Fatal errors MUST use `console.error()` to signal unrecoverable conditions.
- **R-CONSOLE-003** MUST: When catching exceptions, pass the error object as a second argument to console methods to preserve stack traces for debugging.
- **R-CONSOLE-004** SHOULD: Use template literals for message formatting to include contextual information such as file paths, operation names, or component identifiers.
- **R-CONSOLE-005** SHOULD: Consider prefixing messages with component identifiers in square brackets to aid log filtering in complex workflows.
- **R-CONSOLE-006** MUST: No external logging library dependencies are permitted; only native Console API methods are allowed for error and warning logging.

### Verify

```bash
# Discover and inspect source files in agent, CLI, and core utility directories
find . -type f -name '*.js' -o -name '*.ts' | grep -E '(agent|cli|core|util)' | head -20

# Search for all console method calls to verify warn/error usage
grep -r 'console\.(warn|error)' --include='*.js' --include='*.ts' | head -30

# Verify no external logging library imports are present
grep -r "require.*log\|import.*log" --include='*.js' --include='*.ts' --include='package.json' --include='package-lock.json' | grep -v 'console'

# Review error handling paths for warn/error distinction
grep -r 'console\.log\|console\.debug\|console\.info' --include='*.js' --include='*.ts' | grep -i error

# Check dependency manifests for logging libraries
cat package.json | grep -E '"(winston|pino|bunyan|log4js|debug)"'
```

**Accept when:**
- All error and warning logging across agent, CLI, and core layers uses `console.warn()` or `console.error()` exclusively
- No external logging library dependencies are present in dependency manifests
- Error severity distinction is consistent: `console.warn()` for recoverable errors, `console.error()` for fatal errors
- Error objects are passed as second arguments to console methods to preserve stack traces
- No `console.log()`, `console.debug()`, or `console.info()` calls are used for error reporting

<enforcement>
Claude Code MUST NOT skip or defer verification. All error and warning logging must conform to these rules before code is committed.
</enforcement>
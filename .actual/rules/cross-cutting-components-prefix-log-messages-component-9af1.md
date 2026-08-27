# Native Console API for Error and Warning Logging: Components Prefix Log Messages Component Identifiers

These rules are ALWAYS ACTIVE for all error and warning logging code across agent layers, CLI handlers, and core filesystem utilities.

### Rules

- **R-CONSOLE-001** MAY: Components MAY prefix log messages with component identifiers (e.g., '[ruler]') to aid in log filtering and debugging.
- **R-CONSOLE-002** MUST: Use console.warn() for recoverable errors and console.error() for fatal errors exclusively; do not introduce external logging library dependencies.
- **R-CONSOLE-003** SHOULD: Pass the error object as a second argument to console methods to preserve stack traces for debugging.
- **R-CONSOLE-004** SHOULD: Use template literals for message formatting to include contextual information such as file paths, operation names, or component identifiers.

### Verify

```bash
# Discover and inspect source files in agent, CLI, and core utility directories
find . -type f -name '*.js' -o -name '*.ts' | xargs grep -l 'console\.(warn|error)' | head -20

# Search the codebase for console method calls and verify no external logging library imports
grep -r 'require.*log\|import.*log' --include='*.js' --include='*.ts' | grep -v node_modules | grep -v '.actual'

# Review error handling paths to confirm warn/error distinction
grep -r 'console\.warn\|console\.error' --include='*.js' --include='*.ts' | grep -v node_modules | grep -v '.actual'

# Verify dependency manifest contains no logging libraries
cat package.json | grep -E '"(winston|pino|bunyan|log4js|debug)"'
```

**Accept when:**
- All error and warning logging across agent, CLI, and core layers uses console.warn() or console.error() exclusively
- No external logging library dependencies are present in dependency manifests
- Error severity distinction is consistent: console.warn() for recoverable errors, console.error() for fatal errors
- Error objects are passed as second arguments to console methods to preserve stack traces
- Component identifiers in square brackets are used consistently where applicable

<enforcement>
Claude Code MUST NOT skip or defer verification. All error logging must conform to the native Console API pattern before code is accepted.
</enforcement>
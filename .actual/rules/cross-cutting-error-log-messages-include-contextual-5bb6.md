# Native Console API for Error and Warning Logging: Error Log Messages Include Contextual Information

These rules are ALWAYS ACTIVE for all error and warning logging across agent layer, CLI layer, and core utility files.

### Rules

- **R-CONSOLE-001** SHOULD: Error log messages SHOULD include contextual information using template literals for string interpolation.
- **R-CONSOLE-002** SHOULD: Use console.warn for recoverable errors and console.error for fatal errors to maintain consistent severity distinction.
- **R-CONSOLE-003** SHOULD: Pass the error object as a second argument to console methods to preserve stack traces for debugging.
- **R-CONSOLE-004** SHOULD: Consider prefixing messages with component identifiers in square brackets to aid log filtering in complex workflows.

### Verify

```bash
# Discover and inspect source files in agent, CLI, and core utility directories
find . -type f -name '*.js' -o -name '*.ts' | grep -E '(agent|cli|core|util)' | head -20

# Search for all console method calls
grep -r 'console\.(warn|error)' --include='*.js' --include='*.ts' .

# Verify no external logging library imports are present
grep -r "require.*log\|import.*log" --include='*.js' --include='*.ts' . | grep -v node_modules | grep -v '.actual'

# Review error handling patterns to confirm warn/error distinction
grep -r 'console\.warn\|console\.error' --include='*.js' --include='*.ts' . -B 2 -A 2

# Check dependency manifest for logging library entries
cat package.json | grep -i 'log\|winston\|pino\|bunyan'
```

**Accept when:**
- All error and warning logging across agent, CLI, and core layers uses console.warn or console.error exclusively
- No external logging library dependencies are present in dependency manifests
- Error severity distinction is consistent: console.warn for recoverable errors, console.error for fatal errors
- Error objects are passed as second arguments to console methods to preserve stack traces
- Component identifiers are used as message prefixes where applicable

<enforcement>
Claude Code MUST NOT skip or defer verification. All error logging must conform to these rules before code is accepted.
</enforcement>
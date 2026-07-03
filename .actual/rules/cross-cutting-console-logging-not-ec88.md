# Standardize console.warn and console.error for Operational Logging: Console Logging Not

These rules are ALWAYS ACTIVE for all agent implementations, CLI handlers, configuration loaders, and file system utilities within the ruler codebase.

### Rules

- **R-CONSOLE-001** MUST NOT: Console logging MUST NOT be used for structured telemetry, metrics, or audit trails that require persistence or aggregation.
- **R-CONSOLE-002** MUST: Use console.warn for recoverable conditions (missing optional config, fallback behavior).
- **R-CONSOLE-003** MUST: Use console.error for conditions requiring user intervention or critical failures.
- **R-CONSOLE-004** MUST: Include contextual information in all console messages: operation name, file path, and original error message.
- **R-CONSOLE-005** SHOULD: Define constants for common prefixes (e.g., ERROR_PREFIX, WARN_PREFIX) in a shared constants module to ensure consistency across CLI handlers.
- **R-CONSOLE-006** SHOULD: Consider wrapping console calls in utility functions (e.g., logWarning, logError) to centralize formatting logic and enable future migration to structured logging.
- **R-CONSOLE-007** MUST NOT: Use console.log in production code paths; restrict to console.warn and console.error for operational diagnostics only.

### Verify

```bash
# Count console.warn usage in core modules
grep -r 'console\.warn' src/ | grep -E '(ConfigLoader|FirebenderAgent|handlers|FileSystemUtils)' | wc -l

# Count console.error usage in core modules
grep -r 'console\.error' src/ | grep -E '(ConfigLoader|FirebenderAgent|handlers|FileSystemUtils)' | wc -l

# Verify no console.log in production code
grep -r 'console\.log' src/ | grep -v test | wc -l
```

**Accept when:**
- All configuration loading failures and file system errors in core modules use console.warn or console.error appropriately
- CLI handlers consistently use ERROR_PREFIX or similar constants for error messages
- No console.log usage exists in production code paths (only console.warn and console.error for operational diagnostics)
- All console messages include contextual information (operation name, file path, original error)

<enforcement>
Claude Code MUST NOT skip or defer verification. ESLint rules restricting console.log in production code and code review checklists requiring console.warn/console.error usage for operational diagnostics are mandatory.
</enforcement>
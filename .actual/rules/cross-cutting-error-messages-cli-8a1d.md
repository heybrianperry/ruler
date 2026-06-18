# Standardize console.warn and console.error for Operational Logging: Error Messages Cli

These rules are ALWAYS ACTIVE for all agent implementations, CLI handlers, configuration loaders, and file system utilities within the ruler codebase.

### Rules

- **R-CONSOLE-001** SHOULD: Error messages in CLI handlers SHOULD use a consistent ERROR_PREFIX constant to maintain uniform presentation across command failures.
- **R-CONSOLE-002** SHOULD: Use console.warn for recoverable conditions (missing optional config, fallback behavior) and console.error for conditions requiring user intervention.
- **R-CONSOLE-003** SHOULD: Include contextual information in all console messages: operation name, file path, and original error message to aid user troubleshooting.
- **R-CONSOLE-004** MUST: Restrict console.log in production code paths; use only console.warn and console.error for operational diagnostics.

### Verify

```bash
# Count console.warn usage in core modules
grep -r 'console\.warn' src/ | grep -E '(ConfigLoader|FirebenderAgent|handlers|FileSystemUtils)' | wc -l

# Count console.error usage in core modules
grep -r 'console\.error' src/ | grep -E '(ConfigLoader|FirebenderAgent|handlers|FileSystemUtils)' | wc -l

# Verify no console.log in production code (non-test)
grep -r 'console\.log' src/ | grep -v test | wc -l

# Check for ERROR_PREFIX or similar constants in CLI handlers
grep -r 'ERROR_PREFIX\|WARN_PREFIX' src/ | grep -E '(handlers|cli)' | wc -l
```

**Accept when:**
- All configuration loading failures and file system errors in core modules use console.warn or console.error appropriately
- CLI handlers consistently use ERROR_PREFIX or similar constants for error messages
- No console.log usage exists in production code paths (only console.warn and console.error for operational diagnostics)
- All console messages include contextual information (operation name, file path, original error)

<enforcement>
Claude Code MUST NOT skip or defer verification. ESLint rules restricting console.log in production code and code review checklists requiring console.warn/console.error usage are mandatory enforcement mechanisms.
</enforcement>
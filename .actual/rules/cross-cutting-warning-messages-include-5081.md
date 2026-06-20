# Standardize console.warn and console.error for Operational Logging: Warning Messages Include

These rules are ALWAYS ACTIVE for all agent implementations, CLI handlers, configuration loaders, and file system utilities within the ruler codebase.

### Rules

- **R-CONSOLE-001** SHOULD: Warning messages SHOULD include a `[ruler]` prefix to identify the source system and the specific file path or operation that triggered the warning.
- **R-CONSOLE-002** MUST: Use `console.warn` for recoverable conditions (missing optional config, fallback behavior).
- **R-CONSOLE-003** MUST: Use `console.error` for conditions requiring user intervention or critical failures.
- **R-CONSOLE-004** MUST: Include contextual information in all console messages: operation name, file path, and original error message.
- **R-CONSOLE-005** MUST NOT: Use `console.log` in production code paths; restrict to `console.warn` and `console.error` only.
- **R-CONSOLE-006** SHOULD: Define constants for common prefixes (e.g., `ERROR_PREFIX`, `WARN_PREFIX`) in a shared constants module.
- **R-CONSOLE-007** MAY: Wrap console calls in utility functions (e.g., `logWarning`, `logError`) to centralize formatting logic.

### Verify

```bash
# Count console.warn usage in core modules
grep -r 'console\.warn' src/ | grep -E '(ConfigLoader|FirebenderAgent|handlers|FileSystemUtils)' | wc -l

# Count console.error usage in core modules
grep -r 'console\.error' src/ | grep -E '(ConfigLoader|FirebenderAgent|handlers|FileSystemUtils)' | wc -l

# Verify no console.log in production code
grep -r 'console\.log' src/ | grep -v test | wc -l

# Check for [ruler] prefix in warning messages
grep -r 'console\.warn' src/ | grep -E '(ConfigLoader|FirebenderAgent|handlers|FileSystemUtils)' | grep -c '\[ruler\]'
```

**Accept when:**
- All configuration loading failures and file system errors in core modules use `console.warn` or `console.error` appropriately
- CLI handlers consistently use `ERROR_PREFIX` or similar constants for error messages
- No `console.log` usage exists in production code paths (only `console.warn` and `console.error` for operational diagnostics)
- Warning messages include the `[ruler]` prefix with file path or operation context
- Contextual information (operation name, file path, original error) is present in all console messages

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement.
</enforcement>
# Standardize Console-Based Logging with Prefixed Messages: Logging Output Use

These rules are ALWAYS ACTIVE for all TypeScript modules in the src/ directory, including CLI handlers, core infrastructure modules, and utility modules that perform logging or error handling.

### Rules

- **R-LOG-001** MUST: All logging output MUST use native console methods (console.error, console.warn, console.log) without external logging frameworks.
- **R-LOG-002** MUST: All console logging calls MUST use centralized utility functions (logVerbose, logInfo, logWarn, logError) exported from constants.ts or a dedicated logging module.
- **R-LOG-003** MUST: Log messages MUST include consistent prefixes that identify source and severity (e.g., '[ruler]', '[ruler:verbose]', ERROR_PREFIX).
- **R-LOG-004** MUST: Error-level messages MUST use console.error and warning-level messages MUST use console.warn.
- **R-LOG-005** SHOULD: Standard prefixes SHOULD be defined as exported constants to prevent string literal duplication.
- **R-LOG-006** SHOULD: Logging utilities SHOULD be wrapped in functions to enable future enhancements like log level filtering or output redirection.

### Verify

```bash
# Check for direct console method usage outside utility functions (should be <= 5 instances)
grep -r 'console\.(log|error|warn)' src/ | grep -v 'logVerbose\|logInfo\|logWarn' | wc -l | awk '{if ($1 > 5) exit 1}'

# Verify logging utilities are exported from constants.ts
grep -r 'export.*log(Verbose|Info|Warn|Error)' src/constants.ts

# Verify error-level messages use console.error with proper prefix
grep -r 'console\.error.*\[ruler' src/ | wc -l | awk '{if ($1 > 0) exit 0; else exit 1}'
```

**Accept when:**
- All console logging calls use centralized utility functions (logVerbose, logInfo, logWarn, logError) exported from constants.ts or a dedicated logging module
- Log messages include consistent prefixes that identify source and severity
- Error-level messages use console.error and warning-level messages use console.warn
- Direct console method calls outside utility functions do not exceed 5 instances in the codebase
- Standard prefixes are defined as exported constants

<enforcement>
Claude Code MUST NOT skip or defer verification. All logging patterns MUST be checked against centralized utility functions before accepting code changes. Violations detected by grep-based verification commands MUST cause CI pipeline failure and require refactoring to use centralized logging utilities.
</enforcement>
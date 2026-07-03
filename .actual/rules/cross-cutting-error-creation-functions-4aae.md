# Standardize Console-Based Logging with Prefixed Messages: Error Creation Functions

These rules are ALWAYS ACTIVE for all TypeScript modules in the src/ directory, including CLI handlers, core infrastructure modules, and utility modules that perform console-based logging.

### Rules

- **R-LOG-001** SHOULD: Error creation functions (createRulerError) SHOULD be provided as part of the public API contract to standardize error handling across the application.
- **R-LOG-002** MUST: All console logging calls MUST use centralized utility functions (logVerbose, logInfo, logWarn, logError) exported from constants or a dedicated logging module rather than direct console method calls.
- **R-LOG-003** MUST: Log messages MUST include consistent prefixes (e.g., '[ruler]', '[ruler:verbose]', ERROR_PREFIX) that identify source and severity level.
- **R-LOG-004** MUST: Error-level messages MUST use console.error and warning-level messages MUST use console.warn to enable proper stderr/stdout routing.
- **R-LOG-005** SHOULD: Standard prefixes SHOULD be defined as exported constants to prevent string literal duplication and ensure consistency across modules.
- **R-LOG-006** MAY: Exceptions MAY be granted for third-party library integration code that requires specific console usage, provided the exception is documented with inline comments explaining the deviation.

### Verify

```bash
# Check for direct console calls outside utility functions (should be <= 5)
grep -r 'console\.(log|error|warn)' src/ | grep -v 'logVerbose\|logInfo\|logWarn' | wc -l | awk '{if ($1 > 5) exit 1}'

# Verify logging utilities are exported from constants
grep -r 'export.*log(Verbose|Info|Warn|Error)' src/constants.ts

# Verify error-level console calls use proper prefix pattern
grep -r 'console\.error.*\[ruler' src/ | wc -l | awk '{if ($1 > 0) exit 0; else exit 1}'
```

**Accept when:**
- All console logging calls use centralized utility functions (logVerbose, logInfo, logWarn, logError) exported from constants or logging module
- Log messages include consistent prefixes that identify source and severity
- Error-level messages use console.error and warning-level messages use console.warn
- Direct console method calls outside utility functions do not exceed 5 instances
- Standard prefixes are defined as exported constants

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline validation.
</enforcement>
# Standardize Console-Based Logging with Prefixed Messages: Log Messages Include

These rules are ALWAYS ACTIVE for all TypeScript modules in the src/ directory, including CLI handlers, core infrastructure modules, and utility modules that perform console-based logging.

### Rules

- **R-LOG-001** MUST: Log messages MUST include a prefix identifier (e.g., '[ruler]', '[ruler:verbose]', ERROR_PREFIX) to indicate the source and context of the message.
- **R-LOG-002** MUST: All console logging calls MUST use centralized utility functions (logVerbose, logInfo, logWarn, logError) exported from constants or a dedicated logging module.
- **R-LOG-003** MUST: Error-level messages MUST use console.error and warning-level messages MUST use console.warn.
- **R-LOG-004** SHOULD: Standard prefixes SHOULD be defined as exported constants to prevent string literal duplication.
- **R-LOG-005** SHOULD: Logging utilities SHOULD be centralized in src/constants.ts or a dedicated logging module to ensure consistent formatting and prefix usage.

### Verify

```bash
# Check for direct console calls outside utility functions (should be <= 5)
grep -r 'console\.(log|error|warn)' src/ | grep -v 'logVerbose\|logInfo\|logWarn' | wc -l | awk '{if ($1 > 5) exit 1}'

# Verify logging utilities are exported
grep -r 'export.*log(Verbose|Info|Warn|Error)' src/constants.ts

# Verify console.error calls use ruler prefix
grep -r 'console\.error.*\[ruler' src/ | wc -l | awk '{if ($1 > 0) exit 0; else exit 1}'
```

**Accept when:**
- All console logging calls use centralized utility functions (logVerbose, logInfo, logWarn, logError) exported from constants or logging module
- Log messages include consistent prefixes that identify source and severity
- Error-level messages use console.error and warning-level messages use console.warn
- Direct console method usage outside utility functions does not exceed 5 instances
- Standard prefixes are defined as exported constants

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory and must be checked before accepting code changes.
</enforcement>
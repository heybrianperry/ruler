# Standardize Console-Based Logging with Prefixed Messages: Modules Define Module

These rules are ALWAYS ACTIVE for all TypeScript modules in the src/ directory, including CLI handlers, core infrastructure modules, and utility modules that perform console-based logging.

### Rules

- **R-LOG-001** MAY: Modules MAY define module-specific prefixes to distinguish log sources within the application.
- **R-LOG-002** MUST: All console logging calls MUST use centralized utility functions (logVerbose, logInfo, logWarn, logError) exported from constants or a dedicated logging module.
- **R-LOG-003** MUST: Log messages MUST include consistent prefixes that identify source and severity (e.g., '[ruler]', '[ruler:verbose]', ERROR_PREFIX).
- **R-LOG-004** MUST: Error-level messages MUST use console.error and warning-level messages MUST use console.warn.
- **R-LOG-005** SHOULD: Standard prefixes SHOULD be defined as exported constants to prevent string literal duplication.

### Verify

```bash
# Check for direct console method usage outside utility functions
grep -r 'console\.(log|error|warn)' src/ | grep -v 'logVerbose\|logInfo\|logWarn' | wc -l | awk '{if ($1 > 5) exit 1}'

# Verify logging utilities are exported
grep -r 'export.*log(Verbose|Info|Warn|Error)' src/constants.ts

# Verify error-level messages use console.error with prefixes
grep -r 'console\.error.*\[ruler' src/ | wc -l | awk '{if ($1 > 0) exit 0; else exit 1}'
```

**Accept when:**
- All console logging calls use centralized utility functions (logVerbose, logInfo, logWarn) exported from constants or logging module
- Log messages include consistent prefixes that identify source and severity
- Error-level messages use console.error and warning-level messages use console.warn
- Direct console method usage outside utilities does not exceed 5 instances
- Standard prefixes are defined as exported constants

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline validation.
</enforcement>
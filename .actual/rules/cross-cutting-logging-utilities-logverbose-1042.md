# Standardize Console-Based Logging with Prefixed Messages: Logging Utilities Logverbose

These rules are ALWAYS ACTIVE for all TypeScript modules in the src/ directory, including CLI handlers, core infrastructure modules, and utility modules that perform console-based logging.

### Rules

- **R-LOG-001** MUST: All logging output MUST use centralized logging utility functions (logVerbose, logInfo, logWarn, logError) exported from a central constants or utilities module rather than direct console method calls.
- **R-LOG-002** MUST: Log messages MUST include consistent prefixes (e.g., '[ruler]', '[ruler:verbose]', ERROR_PREFIX) that identify the source and severity level of the message.
- **R-LOG-003** SHOULD: Logging utilities (logVerbose, logInfo, logWarn) SHOULD be exported from a central constants or utilities module to ensure consistent formatting across the application.
- **R-LOG-004** MUST: Error-level messages MUST use console.error and warning-level messages MUST use console.warn to enable proper stderr/stdout routing.
- **R-LOG-005** MAY: Environment-based guards MAY be implemented in logging utilities to suppress verbose logs in production environments.

### Verify

```bash
# Check for direct console calls outside utility functions (should be <= 5)
grep -r 'console\.(log|error|warn)' src/ | grep -v 'logVerbose\|logInfo\|logWarn' | wc -l | awk '{if ($1 > 5) exit 1}'

# Verify logging utilities are exported from constants module
grep -r 'export.*log(Verbose|Info|Warn|Error)' src/constants.ts

# Verify console.error calls use ruler prefix
grep -r 'console\.error.*\[ruler' src/ | wc -l | awk '{if ($1 > 0) exit 0; else exit 1}'
```

**Accept when:**
- All console logging calls use centralized utility functions (logVerbose, logInfo, logWarn, logError) exported from constants or logging module
- Log messages include consistent prefixes that identify source and severity
- Error-level messages use console.error and warning-level messages use console.warn
- Direct console method calls outside utility functions do not exceed 5 instances in the codebase

<enforcement>
Claude Code MUST NOT skip or defer verification. All logging patterns MUST be checked against centralized utilities before accepting code changes. CI pipeline MUST fail if direct console calls exceed threshold or if logging utilities are not properly exported.
</enforcement>
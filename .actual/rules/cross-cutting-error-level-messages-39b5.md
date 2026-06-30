# Standardize Console-Based Logging with Prefixed Messages: Error Level Messages

These rules are ALWAYS ACTIVE for all TypeScript modules in the src/ directory, including CLI handlers, core infrastructure modules, and utility modules that perform error handling and diagnostic logging.

### Rules

- **R-LOG-001** MUST: Error-level messages MUST use console.error for stderr output.
- **R-LOG-002** MUST: All console logging calls MUST use centralized utility functions (logVerbose, logInfo, logWarn, logError) exported from constants or a dedicated logging module.
- **R-LOG-003** MUST: Log messages MUST include consistent prefixes that identify source and severity (e.g., '[ruler]', '[ruler:verbose]', ERROR_PREFIX).
- **R-LOG-004** SHOULD: Warning-level messages SHOULD use console.warn for stderr output.
- **R-LOG-005** MAY: Direct console method calls MAY be used in third-party library integration code with inline comments explaining the deviation.

### Verify

```bash
# Check for direct console calls outside utility functions (should be <= 5)
grep -r 'console\.(log|error|warn)' src/ | grep -v 'logVerbose\|logInfo\|logWarn' | wc -l | awk '{if ($1 > 5) exit 1}'

# Verify logging utilities are exported
grep -r 'export.*log(Verbose|Info|Warn|Error)' src/constants.ts

# Verify error-level messages use console.error with prefixes
grep -r 'console\.error.*\[ruler' src/ | wc -l | awk '{if ($1 > 0) exit 0; else exit 1}'
```

**Accept when:**
- All console logging calls use centralized utility functions (logVerbose, logInfo, logWarn, logError) exported from constants or logging module
- Log messages include consistent prefixes that identify source and severity
- Error-level messages use console.error and warning-level messages use console.warn
- Direct console calls outside utilities do not exceed 5 instances
- Any exceptions are marked with inline comments explaining the deviation

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory and must be checked before accepting code changes.
</enforcement>
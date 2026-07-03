# Standardize Console-Based Logging with Prefixed Messages: Prefix Constants Defined

These rules are ALWAYS ACTIVE for all TypeScript/JavaScript files in the codebase that use console-based logging for CLI operations, file system utilities, and core functionality.

### Rules

- **R-PREFIX-001** SHOULD: Prefix constants SHOULD be defined in a central location (constants.ts) and exported for reuse across modules.
- **R-PREFIX-002** MUST: All console logging calls MUST include a prefix identifier indicating source or severity level.
- **R-PREFIX-003** MUST: Centralized logging functions (logVerbose, logInfo, logWarn, logError) MUST be exported from constants.ts and used consistently across modules.
- **R-PREFIX-004** MUST: Error and warning messages MUST use console.error or console.warn to route output to stderr.
- **R-PREFIX-005** SHOULD: Logging prefix constants (ERROR_PREFIX, INFO_PREFIX, etc.) SHOULD follow a consistent format across all modules.

### Verify

```bash
# Check for unprefixed console usage
grep -r "console\.\(error\|warn\|log\)" src/ | grep -v "node_modules" | head -20

# Count prefix usage patterns
grep -r "\[ruler" src/ | grep -v "node_modules" | wc -l

# Verify centralized logging functions are exported
grep "export.*log" src/constants.ts

# Verify no direct console calls without prefixes in production code
grep -r "console\.\(error\|warn\|log\)" src/ | grep -v "node_modules" | grep -v "logVerbose\|logInfo\|logWarn\|logError" | wc -l
```

**Accept when:**
- All console logging calls include a prefix identifier indicating source or severity level
- Centralized logging functions (logVerbose, logInfo, logWarn, logError) are exported from constants.ts and used consistently across modules
- Error and warning messages use console.error or console.warn to route output to stderr
- All logging prefix constants are defined in src/constants.ts and exported for module-wide reuse
- No unprefixed direct console usage exists in production code (excluding third-party integrations and documented exceptions)

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement. Violations MUST be addressed through code review feedback requesting use of centralized logging functions, or CI pipeline warnings for direct console usage without prefixes.
</enforcement>
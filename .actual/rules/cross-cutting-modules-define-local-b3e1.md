# Standardize Console-Based Logging with Prefixed Messages: Modules Define Local

These rules are ALWAYS ACTIVE for all TypeScript/JavaScript source files in the codebase that perform logging or error reporting.

### Rules

- **R-LOG-001** MUST: All console logging calls (console.error, console.warn, console.log) include a prefix identifier indicating source or severity level.
- **R-LOG-002** MUST: Use console.error for error and warning messages, console.log for informational output to maintain proper stderr/stdout separation.
- **R-LOG-003** SHOULD: Centralized logging functions (logVerbose, logInfo, logWarn, logError) exported from constants.ts be used consistently across modules.
- **R-LOG-004** MAY: Modules MAY define local prefix variables for context-specific logging when centralized prefixes are insufficient.

### Verify

```bash
# Check for unprefixed console usage
grep -r "console\.\(error\|warn\|log\)" src/ | grep -v "node_modules" | head -20

# Count prefixed logging instances
grep -r "\[ruler" src/ | grep -v "node_modules" | wc -l

# Verify centralized logging functions are exported
grep "export.*log" src/constants.ts
```

**Accept when:**
- All console logging calls include a prefix identifier indicating source or severity level
- Centralized logging functions (logVerbose, logInfo, logWarn) are exported from constants.ts and used consistently across modules
- Error and warning messages use console.error or console.warn to route output to stderr
- All logging prefix constants (ERROR_PREFIX, INFO_PREFIX, etc.) are defined in src/constants.ts and exported for module-wide reuse

<enforcement>
Claude Code MUST NOT skip or defer verification. All new logging calls must be reviewed against R-LOG-001 through R-LOG-004 before acceptance.
</enforcement>
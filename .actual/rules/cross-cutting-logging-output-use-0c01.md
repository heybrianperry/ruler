# Standardize Console-Based Logging with Prefixed Messages: Logging Output Use

These rules are ALWAYS ACTIVE for all files in the codebase that perform logging or console output, including CLI handlers, utilities, and core modules.

### Rules

- **R-LOG-001** MUST: All logging output MUST use console methods (console.error, console.warn, console.log) as the primary logging mechanism.
- **R-LOG-002** MUST: All console logging calls MUST include a prefix identifier indicating source or severity level.
- **R-LOG-003** MUST: Centralized logging functions (logVerbose, logInfo, logWarn, logError) MUST be exported from src/constants.ts and used consistently across modules.
- **R-LOG-004** MUST: Error and warning messages MUST use console.error or console.warn to route output to stderr.
- **R-LOG-005** MUST: All logging prefix constants (ERROR_PREFIX, INFO_PREFIX, etc.) MUST be defined in src/constants.ts and exported for module-wide reuse.
- **R-LOG-006** SHOULD: Direct console usage without prefixes should be avoided in favor of centralized logging functions.

### Verify

```bash
# Detect unprefixed console usage
grep -r "console\.\(error\|warn\|log\)" src/ | grep -v "node_modules" | head -20

# Count prefixed logging instances
grep -r "\[ruler" src/ | grep -v "node_modules" | wc -l

# Verify centralized logging functions are exported
grep "export.*log" src/constants.ts
```

**Accept when:**
- All console logging calls include a prefix identifier indicating source or severity level
- Centralized logging functions (logVerbose, logInfo, logWarn, logError) are exported from src/constants.ts and used consistently across modules
- Error and warning messages use console.error or console.warn to route output to stderr
- All logging prefix constants are defined in src/constants.ts

<enforcement>
Clause Code MUST NOT skip or defer verification. All logging output must conform to these rules before code review approval.
</enforcement>
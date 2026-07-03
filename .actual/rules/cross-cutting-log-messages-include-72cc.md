# Standardize Console-Based Logging with Prefixed Messages: Log Messages Include

These rules are ALWAYS ACTIVE for all console logging calls across CLI operations, file system utilities, and core functionality modules.

### Rules

- **R-LOG-001** MUST: Log messages MUST include a prefix identifier that indicates the source or severity level (e.g., '[ruler:verbose]', '[ruler]', ERROR_PREFIX).
- **R-LOG-002** MUST: All logging prefix constants (ERROR_PREFIX, INFO_PREFIX, etc.) MUST be defined in src/constants.ts and exported for module-wide reuse.
- **R-LOG-003** MUST: Centralized logging functions (logVerbose, logInfo, logWarn, logError) MUST be implemented in constants.ts and used consistently across modules.
- **R-LOG-004** MUST: Error and warning messages MUST use console.error or console.warn to route output to stderr; informational output MUST use console.log.

### Verify

```bash
# Check for unprefixed console usage
grep -r "console\.\(error\|warn\|log\)" src/ | grep -v "node_modules" | head -20

# Count prefixed log messages
grep -r "\[ruler" src/ | grep -v "node_modules" | wc -l

# Verify centralized logging functions are exported
grep "export.*log" src/constants.ts
```

**Accept when:**
- All console logging calls include a prefix identifier indicating source or severity level
- Centralized logging functions (logVerbose, logInfo, logWarn) are exported from constants.ts and used consistently across modules
- Error and warning messages use console.error or console.warn to route output to stderr
- No direct console usage without prefixes is detected in grep verification

<enforcement>
Claude Code MUST NOT skip or defer verification of these logging rules. All new logging calls must be reviewed against R-LOG-001 through R-LOG-004 before acceptance.
</enforcement>
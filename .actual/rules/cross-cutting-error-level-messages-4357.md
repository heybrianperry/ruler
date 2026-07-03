# Standardize Console-Based Logging with Prefixed Messages: Error Level Messages

These rules are ALWAYS ACTIVE for all files in the codebase that implement console-based logging, including constants.ts, FileSystemUtils.ts, CLI handlers, and any module performing error reporting or diagnostic output.

### Rules

- **R-LOG-ERR-001** MUST: Error-level messages MUST use console.error for output to ensure proper stream routing to stderr.
- **R-LOG-ERR-002** MUST: All console logging calls MUST include a prefix identifier indicating source or severity level (e.g., '[ruler]', '[ruler:verbose]', ERROR_PREFIX).
- **R-LOG-ERR-003** MUST: All logging prefix constants (ERROR_PREFIX, INFO_PREFIX, etc.) MUST be defined in src/constants.ts and exported for module-wide reuse.
- **R-LOG-ERR-004** SHOULD: Centralized logging functions (logVerbose, logInfo, logWarn, logError) SHOULD be implemented in constants.ts and used consistently across modules rather than direct console calls.
- **R-LOG-ERR-005** SHOULD: Warning messages SHOULD use console.warn to route output to stderr and maintain proper stream separation.
- **R-LOG-ERR-006** MAY: Temporary debugging code not intended for merge MAY use direct console output without prefixes if documented with a code comment explaining the exception.

### Verify

```bash
# Detect unprefixed console usage
grep -r "console\.\(error\|warn\|log\)" src/ | grep -v "node_modules" | head -20

# Count prefixed logging instances
grep -r "\[ruler" src/ | grep -v "node_modules" | wc -l

# Verify centralized logging functions are exported
grep "export.*log" src/constants.ts

# Check for console.error usage in error handling paths
grep -r "console\.error" src/ | grep -v "node_modules"
```

**Accept when:**
- All console logging calls include a prefix identifier indicating source or severity level
- Centralized logging functions (logVerbose, logInfo, logWarn, logError) are exported from constants.ts
- Error and warning messages use console.error or console.warn to route output to stderr
- No unprefixed direct console calls exist outside of documented temporary debugging code
- All logging prefix constants are centralized in src/constants.ts

<enforcement>
Claude Code MUST NOT skip or defer verification. All new logging implementations MUST be verified against these rules before acceptance. Code review feedback MUST be provided for violations, and CI pipeline warnings MUST be generated for direct console usage without prefixes.
</enforcement>
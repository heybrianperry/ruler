# Standardize Console-Based Logging with Prefixed Messages: Logging Utilities Centralized

These rules are ALWAYS ACTIVE for all TypeScript/JavaScript files in the codebase that perform logging or console output operations.

### Rules

- **R-LOG-001** MUST: All logging output MUST use centralized logging functions (logVerbose, logInfo, logWarn, logError) exported from constants.ts rather than direct console method calls.
- **R-LOG-002** MUST: All logging messages MUST include a prefix identifier (e.g., '[ruler:verbose]', '[ruler]', ERROR_PREFIX) indicating source or severity level.
- **R-LOG-003** MUST: Error and warning messages MUST use console.error or console.warn to route output to stderr; informational output MUST use console.log.
- **R-LOG-004** SHOULD: Logging prefix constants (ERROR_PREFIX, INFO_PREFIX, VERBOSE_PREFIX, etc.) SHOULD be defined and exported from src/constants.ts for module-wide reuse.
- **R-LOG-005** SHOULD: Centralized logging functions SHOULD encapsulate console methods and prefix application to maintain consistent formatting across all modules.
- **R-LOG-006** MAY: Direct console usage without prefixes MAY be approved for temporary debugging code not intended for merge, provided it is documented with a code comment explaining the exception.

### Verify

```bash
# Detect unprefixed console usage
grep -r "console\.\(error\|warn\|log\)" src/ | grep -v "node_modules" | grep -v "logVerbose\|logInfo\|logWarn\|logError" | head -20

# Count prefixed log messages in codebase
grep -r "\[ruler" src/ | grep -v "node_modules" | wc -l

# Verify centralized logging functions are exported from constants.ts
grep "export.*log" src/constants.ts

# Verify no direct console calls in production code (excluding test files)
grep -r "console\." src/ --include="*.ts" --exclude-dir=node_modules | grep -v "test\|spec" | wc -l
```

**Accept when:**
- All console logging calls include a prefix identifier indicating source or severity level
- Centralized logging functions (logVerbose, logInfo, logWarn, logError) are exported from constants.ts
- Centralized logging functions are used consistently across modules instead of direct console calls
- Error and warning messages use console.error or console.warn to route output to stderr
- Informational messages use console.log for stdout output
- All logging prefix constants are defined and exported from src/constants.ts

<enforcement>
Claude Code MUST NOT skip or defer verification. All logging calls MUST be reviewed to ensure they use centralized functions with appropriate prefixes. Grep-based verification commands MUST be executed to detect unprefixed console usage before code is considered compliant.
</enforcement>
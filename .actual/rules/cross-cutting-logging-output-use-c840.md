# Standardize Console-Based Logging with Prefixed Messages: Logging Output Use

These rules are ALWAYS ACTIVE for all modules performing file system operations, CLI command handlers, core utility and constant definitions, and error reporting across the codebase.

### Rules

- **R-LOG-001** MUST: All logging output MUST use native console APIs (console.error, console.warn, console.log) without external logging frameworks.
- **R-LOG-002** MUST: All console logging calls in core modules (constants.ts, FileSystemUtils.ts, handlers.ts) MUST include a descriptive prefix (e.g., [ruler], [ruler:verbose], ${ERROR_PREFIX}).
- **R-LOG-003** MUST: Import logging functions (logVerbose, logInfo, logWarn) from constants.ts rather than calling console methods directly.
- **R-LOG-004** SHOULD: Use template literals with prefix constants for consistent message formatting: `${prefix} ${message}`.
- **R-LOG-005** SHOULD: Reserve console.error for error conditions, console.warn for warnings, and verbose logging for diagnostic output.

### Verify

```bash
# Check for prefixed console calls in core modules
grep -r "console\.error\|console\.warn\|console\.log" src/ --include="*.ts" | grep -E "\[ruler|ERROR_PREFIX|\$\{prefix\}"

# Verify centralized logging functions are exported and used
grep -r "logVerbose\|logInfo\|logWarn" src/ --include="*.ts"

# Confirm logging functions are exported from constants.ts
grep "export.*log" src/constants.ts
```

**Accept when:**
- All console logging calls in src/ include a descriptive prefix (e.g., [ruler], [ruler:verbose], ${ERROR_PREFIX})
- Centralized logging functions (logVerbose, logInfo, logWarn) are exported from constants.ts and used consistently
- No direct console calls exist without prefix formatting in core modules (constants.ts, FileSystemUtils.ts, handlers.ts)
- Logging output is reserved for appropriate severity levels (error, warn, verbose)

<enforcement>
Clause Code MUST NOT skip or defer verification of logging patterns and prefix usage in core modules.
</enforcement>
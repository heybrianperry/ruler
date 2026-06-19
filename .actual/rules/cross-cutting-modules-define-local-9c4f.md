# Standardize Console-Based Logging with Prefixed Messages: Modules Define Local

These rules are ALWAYS ACTIVE for all modules performing file system operations, CLI command handlers, core utility and constant definitions, and error reporting across the codebase.

### Rules

- **R-LOG-001** MAY: Modules MAY define local prefix constants (e.g., ERROR_PREFIX) for consistent message formatting within their scope.
- **R-LOG-002** MUST: Import logging functions (logVerbose, logInfo, logWarn) from constants.ts rather than calling console methods directly in core modules.
- **R-LOG-003** MUST: Use template literals with prefix constants for consistent message formatting: `${prefix} ${message}`.
- **R-LOG-004** SHOULD: Reserve console.error for error conditions, console.warn for warnings, and verbose logging for diagnostic output.
- **R-LOG-005** MUST: All console logging calls in core modules (constants.ts, FileSystemUtils.ts, handlers.ts) include a descriptive prefix (e.g., [ruler], [ruler:verbose], ${ERROR_PREFIX}).

### Verify

```bash
# Check for prefixed console calls in core modules
grep -r "console\.error\|console\.warn\|console\.log" src/ --include="*.ts" | grep -E "\[ruler|ERROR_PREFIX|\$\{prefix\}"

# Verify centralized logging functions are exported from constants.ts
grep "export.*log" src/constants.ts

# Verify centralized logging functions are used consistently
grep -r "logVerbose\|logInfo\|logWarn" src/ --include="*.ts"

# Check for unprefixed console calls in core modules (should return empty)
grep -r "console\.error\|console\.warn\|console\.log" src/constants.ts src/FileSystemUtils.ts src/handlers.ts --include="*.ts" | grep -v -E "\[ruler|ERROR_PREFIX|\$\{prefix\}"
```

**Accept when:**
- All console logging calls in src/ include a descriptive prefix (e.g., [ruler], [ruler:verbose], ${ERROR_PREFIX})
- Centralized logging functions (logVerbose, logInfo, logWarn) are exported from constants.ts and used consistently
- No direct console calls exist without prefix formatting in core modules (constants.ts, FileSystemUtils.ts, handlers.ts)
- Grep-based verification commands return expected results with no unprefixed console calls in core modules

<enforcement>
Claude Code MUST NOT skip or defer verification. All console logging calls in core modules must be verified to include proper prefixes before accepting changes.
</enforcement>
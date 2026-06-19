# Standardize Console-Based Logging with Prefixed Messages: Verbose Diagnostic Logging

These rules are ALWAYS ACTIVE for all TypeScript modules performing file system operations, CLI command handlers, and core utility definitions that emit console-based logging output.

### Rules

- **R-LOG-001** SHOULD: Verbose or diagnostic logging SHOULD use a `[ruler:verbose]` prefix to distinguish detailed operational output from standard messages.
- **R-LOG-002** MUST: Import logging functions (logVerbose, logInfo, logWarn) from constants.ts rather than calling console methods directly in core modules.
- **R-LOG-003** MUST: All console logging calls in core modules (constants.ts, FileSystemUtils.ts, handlers.ts) MUST include a descriptive prefix (e.g., [ruler], [ruler:verbose], ${ERROR_PREFIX}).
- **R-LOG-004** SHOULD: Use template literals with prefix constants for consistent message formatting: `${prefix} ${message}`.
- **R-LOG-005** SHOULD: Reserve console.error for error conditions, console.warn for warnings, and verbose logging for diagnostic output.

### Verify

```bash
# Check for unprefixed console calls in core modules
grep -r "console\.error\|console\.warn\|console\.log" src/ --include="*.ts" | grep -v -E "\[ruler|ERROR_PREFIX|\$\{prefix\}" | grep -E "(constants|FileSystemUtils|handlers)\.ts"

# Verify centralized logging functions are exported from constants.ts
grep "export.*log" src/constants.ts | grep -E "logVerbose|logInfo|logWarn"

# Check for consistent use of centralized logging functions
grep -r "logVerbose\|logInfo\|logWarn" src/ --include="*.ts"

# Verify prefix patterns in console calls
grep -r "console\.error\|console\.warn\|console\.log" src/ --include="*.ts" | grep -E "\[ruler|ERROR_PREFIX|\$\{prefix\}"
```

**Accept when:**
- All console logging calls in src/ include a descriptive prefix (e.g., [ruler], [ruler:verbose], ${ERROR_PREFIX})
- Centralized logging functions (logVerbose, logInfo, logWarn) are exported from constants.ts
- Centralized logging functions are used consistently across core modules (constants.ts, FileSystemUtils.ts, handlers.ts)
- No direct console calls exist without prefix formatting in core modules
- Template literals with prefix constants are used for consistent message formatting

<enforcement>
Claude Code MUST NOT skip or defer verification of these logging rules. All console-based logging in scope MUST conform to the prefix and centralization requirements before code is considered compliant.
</enforcement>
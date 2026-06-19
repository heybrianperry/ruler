# Standardize Console-Based Logging with Prefixed Messages: Logging Functions Logverbose

These rules are ALWAYS ACTIVE for all files matching the configured scope: file system utilities (FileSystemUtils.ts), CLI command handlers (handlers.ts), and core utility and constant definitions (constants.ts).

### Rules

- **R-LOG-001** SHOULD: Logging functions (logVerbose, logInfo, logWarn) SHOULD be defined in a central constants module and exported as part of the public API contract.
- **R-LOG-002** MUST: All console logging calls in core modules MUST include a descriptive prefix (e.g., [ruler], [ruler:verbose], ${ERROR_PREFIX}).
- **R-LOG-003** MUST: Import logging functions (logVerbose, logInfo, logWarn) from constants.ts rather than calling console methods directly.
- **R-LOG-004** SHOULD: Use template literals with prefix constants for consistent message formatting: `${prefix} ${message}`.
- **R-LOG-005** SHOULD: Reserve console.error for error conditions, console.warn for warnings, and verbose logging for diagnostic output.

### Verify

```bash
# Check for unprefixed console calls in core modules
grep -r "console\.error\|console\.warn\|console\.log" src/ --include="*.ts" | grep -v -E "\[ruler|ERROR_PREFIX|\$\{prefix\}"

# Verify centralized logging functions are exported from constants.ts
grep "export.*logVerbose\|export.*logInfo\|export.*logWarn" src/constants.ts

# Check for consistent use of centralized logging functions
grep -r "logVerbose\|logInfo\|logWarn" src/ --include="*.ts"
```

**Accept when:**
- All console logging calls in src/ include a descriptive prefix (e.g., [ruler], [ruler:verbose], ${ERROR_PREFIX})
- Centralized logging functions (logVerbose, logInfo, logWarn) are exported from constants.ts and used consistently
- No direct console calls exist without prefix formatting in core modules (constants.ts, FileSystemUtils.ts, handlers.ts)

<enforcement>
Claude Code MUST NOT skip or defer verification. All logging statements in scope must be verified against the rules before acceptance.
</enforcement>
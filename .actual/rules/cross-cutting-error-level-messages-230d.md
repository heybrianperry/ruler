# Standardize Console-Based Logging with Prefixed Messages: Error Level Messages

These rules are ALWAYS ACTIVE for all files matching the configured scope: file system utilities (FileSystemUtils.ts), CLI command handlers (handlers.ts), and core utility and constant definitions (constants.ts).

### Rules

- **R-LOG-001** MUST: Error-level messages MUST use console.error with a descriptive prefix indicating the component or severity (e.g., `[ruler]`, `${ERROR_PREFIX}`).
- **R-LOG-002** MUST: Import logging functions (logVerbose, logInfo, logWarn) from constants.ts rather than calling console methods directly in core modules.
- **R-LOG-003** MUST: Use template literals with prefix constants for consistent message formatting: `${prefix} ${message}`.
- **R-LOG-004** SHOULD: Reserve console.error for error conditions, console.warn for warnings, and verbose logging for diagnostic output.
- **R-LOG-005** SHOULD: Document expected log prefixes ([ruler], [ruler:verbose], ERROR_PREFIX) in module-level comments for maintainability.

### Verify

```bash
# Check for console.error and console.warn calls with descriptive prefixes
grep -r "console\.error\|console\.warn\|console\.log" src/ --include="*.ts" | grep -E "\[ruler|ERROR_PREFIX|\$\{prefix\}"

# Verify centralized logging functions are exported from constants.ts
grep "export.*log" src/constants.ts

# Check for usage of centralized logging functions
grep -r "logVerbose\|logInfo\|logWarn" src/ --include="*.ts"

# Identify any unprefixed console calls in core modules
grep -r "console\.error\|console\.warn\|console\.log" src/constants.ts src/FileSystemUtils.ts src/handlers.ts --include="*.ts" | grep -v -E "\[ruler|ERROR_PREFIX|\$\{prefix\}"
```

**Accept when:**
- All console logging calls in src/ include a descriptive prefix (e.g., [ruler], [ruler:verbose], ${ERROR_PREFIX})
- Centralized logging functions (logVerbose, logInfo, logWarn) are exported from constants.ts and used consistently
- No direct console calls exist without prefix formatting in core modules (constants.ts, FileSystemUtils.ts, handlers.ts)
- Error-level messages specifically use console.error with appropriate prefixes

<enforcement>
Claude Code MUST NOT skip or defer verification. All console logging in scope must be verified to use prefixed messages before accepting changes.
</enforcement>
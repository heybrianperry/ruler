# Standardize Console-Based Logging with Prefixed Messages: Warning Level Messages

These rules are ALWAYS ACTIVE for all modules performing file system operations, CLI command handlers, core utility and constant definitions, and error reporting across the codebase.

### Rules

- **R-LOG-001** MUST: Warning-level messages MUST use console.warn with a prefix indicating the message source.

### Verify

```bash
# Check for console.warn calls with prefixes in core modules
grep -r "console\.warn" src/ --include="*.ts" | grep -E "\[ruler|ERROR_PREFIX|\$\{prefix\}"

# Verify centralized logging functions are exported from constants.ts
grep "export.*logWarn" src/constants.ts

# Check for unprefixed console.warn calls in core modules (should return empty)
grep -r "console\.warn" src/constants.ts src/FileSystemUtils.ts src/handlers.ts --include="*.ts" | grep -v -E "\[ruler|ERROR_PREFIX|\$\{prefix\}"

# Verify usage of centralized logging functions
grep -r "logWarn" src/ --include="*.ts"
```

**Accept when:**
- All console.warn calls in src/ include a descriptive prefix (e.g., [ruler], [ruler:verbose], ${ERROR_PREFIX})
- Centralized logging function logWarn is exported from constants.ts
- logWarn is used consistently across core modules (constants.ts, FileSystemUtils.ts, handlers.ts)
- No direct console.warn calls exist without prefix formatting in core modules

<enforcement>
Claude Code MUST NOT skip or defer verification of this rule. All warning-level logging MUST conform to the prefixed console.warn pattern before code is accepted.
</enforcement>
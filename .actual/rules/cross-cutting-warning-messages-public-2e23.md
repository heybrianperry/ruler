# Standardize Console-Based Logging for Public API Error Reporting: Warning Messages Public

These rules are ALWAYS ACTIVE for all files matching the configured scope: public API exports in src/constants.ts, file system utility functions in src/core/FileSystemUtils.ts, and CLI handler functions in src/cli/handlers.ts.

### Rules

- **R-LOG-001** MUST: Warning messages in public APIs MUST use console.warn with consistent prefix formatting (e.g., '${prefix} ${message}').
- **R-LOG-002** MUST: All console.error and console.warn calls in public API files use consistent prefix formatting.
- **R-LOG-003** MUST: Exported logging functions (logVerbose, logInfo, logWarn) are present in src/constants.ts and used by CLI handlers.
- **R-LOG-004** MUST: CLI error messages are formatted through formatCliError or equivalent formatting function before console output.
- **R-LOG-005** SHOULD: Use consistent prefix patterns: '[ruler:verbose]' for verbose logs, '[ruler]' for standard logs, and ERROR_PREFIX constant for error messages.
- **R-LOG-006** SHOULD: Include contextual information (file paths, operation names, error objects) in error logs to support debugging without requiring verbose mode.

### Verify

```bash
# Check for direct console usage outside approved patterns
grep -r 'console\.error\|console\.warn\|console\.log' src/ --include='*.ts' | grep -v 'logVerbose\|logInfo\|logWarn\|formatCliError'

# Verify logging exports are present in constants.ts
grep -E '(logVerbose|logInfo|logWarn|formatCliError)' src/constants.ts

# Validate logging exports can be loaded
node -e "require('./dist/constants.js'); console.log('Logging exports validated')"
```

**Accept when:**
- All console.error and console.warn calls in public API files use consistent prefix formatting
- Exported logging functions (logVerbose, logInfo, logWarn) are present in src/constants.ts and used by CLI handlers
- CLI error messages are formatted through formatCliError or equivalent formatting function before console output
- No direct console calls are found outside of exported logging functions or formatCliError in public API scope

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code in the specified scope.
</enforcement>
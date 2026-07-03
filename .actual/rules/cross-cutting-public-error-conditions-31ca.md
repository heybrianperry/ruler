# Standardize Console-Based Logging for Public API Error Reporting: Public Error Conditions

These rules are ALWAYS ACTIVE for all public API surfaces and CLI handlers in src/constants.ts, src/core/FileSystemUtils.ts, and src/cli/handlers.ts.

### Rules

- **R-CONSOLE-001** MUST: All public API error conditions MUST be reported using console.error with a descriptive prefix identifying the module or operation context.
- **R-CONSOLE-002** MUST: All exported logging functions (logVerbose, logInfo, logWarn) MUST be centralized in src/constants.ts to provide a single import point for all logging operations.
- **R-CONSOLE-003** MUST: CLI handlers MUST format all user-facing errors through formatCliError or equivalent formatting function before console output to maintain consistent error presentation.
- **R-CONSOLE-004** MUST: Use consistent prefix patterns: '[ruler:verbose]' for verbose logs, '[ruler]' for standard logs, and ERROR_PREFIX constant for error messages.
- **R-CONSOLE-005** SHOULD: Include contextual information (file paths, operation names, error objects) in error logs to support debugging without requiring verbose mode.
- **R-CONSOLE-006** SHOULD: Capture references to console methods at module initialization time to mitigate interference from third-party library console modifications.

### Verify

```bash
# Check for direct console usage outside approved patterns
grep -r 'console\.error\|console\.warn\|console\.log' src/ --include='*.ts' | grep -v 'logVerbose\|logInfo\|logWarn\|formatCliError'

# Verify logging exports are present in constants.ts
grep -E '(logVerbose|logInfo|logWarn|formatCliError)' src/constants.ts

# Validate logging exports can be loaded
node -e "require('./dist/constants.js'); console.log('Logging exports validated')"

# Check for consistent prefix formatting in error logs
grep -r 'console\.error' src/ --include='*.ts' | grep -E '\[ruler|ERROR_PREFIX'
```

**Accept when:**
- All console.error and console.warn calls in public API files use consistent prefix formatting
- Exported logging functions (logVerbose, logInfo, logWarn) are present in src/constants.ts and used by CLI handlers
- CLI error messages are formatted through formatCliError or equivalent formatting function before console output
- No direct console calls exist outside of exported logging functions or formatCliError in public API scope
- Contextual information (file paths, operation names, error objects) is included in error logs

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code in the specified scope. Violations block merge and require exception approval from at least one maintainer.
</enforcement>
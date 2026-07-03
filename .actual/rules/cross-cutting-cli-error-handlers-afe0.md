# Standardize Console-Based Logging for Public API Error Reporting: Cli Error Handlers

These rules are ALWAYS ACTIVE for all files in the public API surfaces and CLI handler boundaries: src/constants.ts, src/core/FileSystemUtils.ts, and src/cli/handlers.ts.

### Rules

- **R-CLI-LOG-001** MUST: CLI error handlers MUST format error messages through dedicated formatting functions (e.g., formatCliError) before console output.
- **R-CLI-LOG-002** MUST: All console.error and console.warn calls in public API files MUST use consistent prefix formatting.
- **R-CLI-LOG-003** MUST: Exported logging functions (logVerbose, logInfo, logWarn) MUST be present in src/constants.ts and used by CLI handlers.
- **R-CLI-LOG-004** MUST: Error messages MUST be formatted through formatCliError or equivalent formatting function before console output.
- **R-CLI-LOG-005** SHOULD: Use consistent prefix patterns: '[ruler:verbose]' for verbose logs, '[ruler]' for standard logs, and ERROR_PREFIX constant for error messages.
- **R-CLI-LOG-006** SHOULD: Include contextual information (file paths, operation names, error objects) in error logs to support debugging without requiring verbose mode.

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
- No direct console calls are found outside of exported logging functions or formatCliError in src/constants.ts, src/core/FileSystemUtils.ts, and src/cli/handlers.ts
- Grep-based verification commands return no violations
- Manual testing of CLI error scenarios verifies consistent error message formatting

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep commands MUST pass with no matches for direct console usage outside approved patterns. All exported logging functions MUST be present and loadable. Code review MUST block merge if violations are detected.
</enforcement>
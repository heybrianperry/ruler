# Standardize Console-Based Logging for Public API Error Reporting: Verbose Logging Utilities

These rules are ALWAYS ACTIVE for all files in src/constants.ts, src/core/FileSystemUtils.ts, and src/cli/handlers.ts that expose public API surfaces and CLI error reporting.

### Rules

- **R-LOG-001** MUST: Verbose logging utilities MUST provide configurable log levels through exported functions (logVerbose, logInfo, logWarn) rather than direct console calls in public API surfaces.
- **R-LOG-002** MUST: All console.error and console.warn calls in public API files MUST use consistent prefix formatting ([ruler:verbose], [ruler], or ERROR_PREFIX constant).
- **R-LOG-003** MUST: CLI error messages MUST be formatted through formatCliError or equivalent formatting function before console output to maintain consistent error presentation.
- **R-LOG-004** SHOULD: Logging function exports SHOULD be centralized in src/constants.ts to provide a single import point for all logging operations.
- **R-LOG-005** SHOULD: Error logs SHOULD include contextual information (file paths, operation names, error objects) to support debugging without requiring verbose mode.
- **R-LOG-006** MAY: Internal debug logging not exposed through public APIs MAY use alternative patterns if documented and isolated from public exports.

### Verify

```bash
# Check for direct console usage outside approved patterns
grep -r 'console\.error\|console\.warn\|console\.log' src/ --include='*.ts' | grep -v 'logVerbose\|logInfo\|logWarn\|formatCliError'

# Verify logging exports are present in constants.ts
grep -E '(logVerbose|logInfo|logWarn|formatCliError)' src/constants.ts

# Validate logging exports can be imported
node -e "require('./dist/constants.js'); console.log('Logging exports validated')"

# Check for consistent prefix patterns in logging calls
grep -r '\[ruler' src/ --include='*.ts' | grep -E '(logVerbose|logInfo|logWarn|ERROR_PREFIX)'
```

**Accept when:**
- All console.error and console.warn calls in public API files (src/constants.ts, src/core/FileSystemUtils.ts, src/cli/handlers.ts) use consistent prefix formatting
- Exported logging functions (logVerbose, logInfo, logWarn) are present in src/constants.ts and used by CLI handlers
- CLI error messages are formatted through formatCliError or equivalent formatting function before console output
- No direct console calls exist outside of exported logging functions or formatCliError in public API scope
- Error logs include contextual information (file paths, operation names, error objects) where applicable

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep-based verification commands MUST pass before accepting changes to logging patterns. Code review MUST enforce prefix consistency for all console logging calls in scope. CI pipeline MUST fail if direct console calls are found outside approved patterns.
</enforcement>
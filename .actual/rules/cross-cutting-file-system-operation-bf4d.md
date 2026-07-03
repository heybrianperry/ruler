# Standardize Console-Based Logging for Public API Error Reporting: File System Operation

These rules are ALWAYS ACTIVE for all files in src/constants.ts, src/core/FileSystemUtils.ts, and src/cli/handlers.ts that expose public API surfaces, file system utilities, and CLI error formatting functions.

### Rules

- **R-LOG-001** SHOULD: File system operation errors SHOULD include contextual information such as directory paths and operation type in error messages.
- **R-LOG-002** MUST: All console.error and console.warn calls in public API files MUST use consistent prefix formatting (e.g., '[ruler:error]', '[ruler:warn]', '[ruler:verbose]').
- **R-LOG-003** MUST: Exported logging functions (logVerbose, logInfo, logWarn) MUST be centralized in src/constants.ts and used by CLI handlers instead of direct console calls.
- **R-LOG-004** MUST: CLI error messages MUST be formatted through formatCliError or equivalent formatting function before console output to maintain consistent error presentation.
- **R-LOG-005** SHOULD: Error logs SHOULD include contextual information (file paths, operation names, error objects) to support debugging without requiring verbose mode.

### Verify

```bash
# Check for direct console usage outside approved patterns
grep -r 'console\.error\|console\.warn\|console\.log' src/ --include='*.ts' | grep -v 'logVerbose\|logInfo\|logWarn\|formatCliError'

# Verify logging exports are present in constants.ts
grep -E '(logVerbose|logInfo|logWarn|formatCliError)' src/constants.ts

# Validate logging exports can be loaded
node -e "require('./dist/constants.js'); console.log('Logging exports validated')"

# Check for consistent prefix formatting in logging calls
grep -r '\[ruler' src/ --include='*.ts' | grep -E '(console\.(error|warn)|logVerbose|logInfo|logWarn)'
```

**Accept when:**
- All console.error and console.warn calls in public API files use consistent prefix formatting
- Exported logging functions (logVerbose, logInfo, logWarn) are present in src/constants.ts and used by CLI handlers
- CLI error messages are formatted through formatCliError or equivalent formatting function before console output
- No direct console calls are found outside of exported logging functions or formatCliError in src/constants.ts, src/core/FileSystemUtils.ts, and src/cli/handlers.ts
- File system operation error messages include contextual information such as directory paths and operation type

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep-based verification commands MUST pass before accepting changes to logging patterns. Code review MUST block merge if error messages lack required prefix formatting or contextual information. CI pipeline MUST fail if direct console calls are found outside approved patterns.
</enforcement>
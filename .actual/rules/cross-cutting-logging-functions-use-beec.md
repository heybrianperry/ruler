# Standardize Console-Based Logging for Public API Error Reporting: Logging Functions Use

These rules are ALWAYS ACTIVE for all files matching the configured scope: public API exports in src/constants.ts, file system utility functions in src/core/FileSystemUtils.ts, and CLI handler functions in src/cli/handlers.ts.

### Rules

- **R-LOG-001** MAY: Logging functions MAY use string interpolation or template literals for message composition to include dynamic runtime values.

### Verify

```bash
# Check for direct console usage outside approved patterns
grep -r 'console\.error\|console\.warn\|console\.log' src/ --include='*.ts' | grep -v 'logVerbose\|logInfo\|logWarn\|formatCliError'

# Verify logging exports are present in constants.ts
grep -E '(logVerbose|logInfo|logWarn|formatCliError)' src/constants.ts

# Validate logging exports can be imported
node -e "require('./dist/constants.js'); console.log('Logging exports validated')"
```

**Accept when:**
- All console.error and console.warn calls in public API files use consistent prefix formatting
- Exported logging functions (logVerbose, logInfo, logWarn) are present in src/constants.ts and used by CLI handlers
- CLI error messages are formatted through formatCliError or equivalent formatting function before console output
- No direct console calls exist outside of exported logging functions or formatCliError in scope files

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST fail if direct console calls are found outside approved patterns. Code review MUST block merge if error messages lack required prefix formatting or contextual information.
</enforcement>
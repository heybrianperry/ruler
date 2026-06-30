# Standardize console.error for Error Logging in Configuration and Environment Management: Error Logging Configuration

These rules are ALWAYS ACTIVE for error logging in configuration and environment management modules, particularly src/core/FileSystemUtils.ts and src/cli/handlers.ts, where file system operations, directory validation, and CLI command execution require lightweight, dependency-free error reporting during runtime initialization.

### Rules

- **R-ERRLOG-001** MUST: Error logging in configuration and environment management modules MUST use console.error for reporting failures in file system operations, directory validation, and environment variable access.

### Verify

```bash
# Verify console.error usage with contextual prefixes in configuration and CLI modules
grep -r 'console\.error' src/core/FileSystemUtils.ts src/cli/handlers.ts | grep -E '(\[ruler\]|ERROR_PREFIX)'

# Verify no external logging library imports in restricted modules
grep -r 'import.*logger' src/core/FileSystemUtils.ts src/cli/handlers.ts && exit 1 || exit 0

# Verify error logging is functional
node -e "require('./src/core/FileSystemUtils.ts'); require('./src/cli/handlers.ts')" 2>&1 | grep -q 'Error' && echo 'Error logging functional' || echo 'No errors detected'
```

**Accept when:**
- All error logging in src/core/FileSystemUtils.ts and src/cli/handlers.ts uses console.error with contextual prefixes (e.g., '[ruler]', ERROR_PREFIX constants)
- No external logging library imports (logger, pino, winston, etc.) are present in configuration and environment management modules
- Error messages include sufficient context (file paths, directory names, operation types) to enable debugging without additional tooling
- console.error calls occur before any process.exit() invocations to guarantee error visibility
- Error messages are consistent in format and include actionable guidance for remediation

<enforcement>
Claude Code MUST NOT skip or defer verification of console.error usage in configuration and CLI modules. All three verify commands MUST pass before accepting changes to src/core/FileSystemUtils.ts and src/cli/handlers.ts. Pull requests introducing external logging dependencies in these restricted modules MUST be rejected with explanation.
</enforcement>
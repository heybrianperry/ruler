# Standardize console.error for Error Logging in Configuration and Environment Management: Error Logging Occur

These rules are ALWAYS ACTIVE for error logging in configuration directory resolution, CLI operations, and environment management modules that interact with file system operations and process environment variables during runtime initialization.

### Rules

- **R-CONSOLE-ERROR-001** MUST: Error logging MUST occur synchronously during configuration resolution and CLI initialization to ensure errors are visible before process termination.
- **R-CONSOLE-ERROR-002** MUST: Use console.error for error reporting in src/core/FileSystemUtils.ts and src/cli/handlers.ts without external logging library dependencies.
- **R-CONSOLE-ERROR-003** MUST: Include consistent error message prefixes (e.g., '[ruler]', ERROR_PREFIX constants) to enable grep-based filtering and identification of error sources.
- **R-CONSOLE-ERROR-004** MUST: Ensure console.error calls occur before any process.exit() invocations to guarantee error visibility.
- **R-CONSOLE-ERROR-005** SHOULD: Include actionable context in error messages: file paths, directory names, operation types, and suggested remediation steps.
- **R-CONSOLE-ERROR-006** SHOULD: Maintain clear, simple English error messages with actionable guidance for end users.

### Verify

```bash
# Verify console.error usage with contextual prefixes in configuration and CLI modules
grep -r 'console\.error' src/core/FileSystemUtils.ts src/cli/handlers.ts | grep -E '(\[ruler\]|ERROR_PREFIX)'

# Verify no external logging library imports in restricted modules
grep -r 'import.*logger' src/core/FileSystemUtils.ts src/cli/handlers.ts && exit 1 || exit 0

# Verify error logging is functional during module initialization
node -e "require('./src/core/FileSystemUtils.ts'); require('./src/cli/handlers.ts')" 2>&1 | grep -q 'Error' && echo 'Error logging functional' || echo 'No errors detected'
```

**Accept when:**
- All error logging in src/core/FileSystemUtils.ts and src/cli/handlers.ts uses console.error with contextual prefixes
- No external logging library imports (logger, pino, winston, etc.) are present in configuration and environment management modules
- Error messages include sufficient context (paths, operation names, directory names) to enable debugging without additional tooling
- console.error calls are positioned before process.exit() invocations to guarantee visibility
- Error messages use consistent prefixes for grep-based filtering and source identification

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for pull requests modifying configuration and CLI error handling. Violations must be addressed before merge.
</enforcement>
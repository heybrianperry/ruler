# Standardize console.error for Error Logging in Configuration and Environment Management: Error Messages Logged

These rules are ALWAYS ACTIVE for error logging in configuration and environment management modules, particularly src/core/FileSystemUtils.ts and src/cli/handlers.ts, where lightweight error reporting during runtime initialization is required.

### Rules

- **R-CONSOLE-001** MUST: Error messages logged via console.error MUST include contextual information such as file paths, directory names, or operation identifiers (e.g., '[ruler] Error checking global config directory').

### Verify

```bash
# Verify console.error usage includes contextual prefixes
grep -r 'console\.error' src/core/FileSystemUtils.ts src/cli/handlers.ts | grep -E '(\[ruler\]|ERROR_PREFIX)'

# Verify no external logging library imports in configuration/CLI modules
grep -r 'import.*logger' src/core/FileSystemUtils.ts src/cli/handlers.ts && exit 1 || exit 0

# Verify error logging is functional
node -e "require('./src/core/FileSystemUtils.ts'); require('./src/cli/handlers.ts')" 2>&1 | grep -q 'Error' && echo 'Error logging functional' || echo 'No errors detected'
```

**Accept when:**
- All error logging in src/core/FileSystemUtils.ts and src/cli/handlers.ts uses console.error with contextual prefixes
- No external logging library imports are present in configuration and environment management modules
- Error messages include sufficient context (paths, operation names) to enable debugging without additional tooling
- Error messages appear before any process.exit() invocations to guarantee visibility

<enforcement>
Claude Code MUST NOT skip or defer verification of console.error usage patterns and contextual prefix presence in configuration and CLI modules.
</enforcement>
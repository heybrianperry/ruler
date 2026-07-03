# Standardize console.error for Error Logging in Configuration and Environment Management: Configuration Environment Management

These rules are ALWAYS ACTIVE for all files in configuration and environment management modules, specifically src/core/FileSystemUtils.ts and src/cli/handlers.ts, where lightweight error reporting during initialization phases is required without external logging dependencies.

### Rules

- **R-CONFIG-001** MUST: Use console.error for error logging in configuration and environment management modules (src/core/FileSystemUtils.ts, src/cli/handlers.ts) during initialization phases.
- **R-CONFIG-002** MUST NOT: Introduce external logging library dependencies (e.g., pino, winston) for error reporting in configuration and environment management modules.
- **R-CONFIG-003** SHOULD: Include contextual prefixes (e.g., '[ruler]', ERROR_PREFIX constants) in all console.error calls to enable grep-based filtering and error source identification.
- **R-CONFIG-004** SHOULD: Include actionable context in error messages: file paths, directory names, operation types, and suggested remediation steps.
- **R-CONFIG-005** MUST: Ensure console.error calls occur before any process.exit() invocations to guarantee error visibility.
- **R-CONFIG-006** SHOULD NOT: Use console.error for structured logging, informational messages, or debug output outside the scope of configuration and CLI error reporting.

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
- All error logging in src/core/FileSystemUtils.ts and src/cli/handlers.ts uses console.error with contextual prefixes
- No external logging library imports are present in configuration and environment management modules
- Error messages include sufficient context (paths, operation names, directory names) to enable debugging without additional tooling
- console.error calls are positioned before any process.exit() invocations
- Error messages are clear, actionable, and include suggested remediation steps where applicable

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All pull requests modifying src/core/FileSystemUtils.ts or src/cli/handlers.ts MUST pass the verify commands above. Violations include: (1) introducing external logging imports in restricted modules, (2) missing contextual prefixes in console.error calls, (3) insufficient error context, or (4) console.error calls occurring after process.exit(). Exceptions require architectural review demonstrating that console.error is insufficient and documenting initialization ordering and dependency implications.
</enforcement>
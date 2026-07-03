# Standardize console.error for Error Logging in Configuration and Environment Management: Cli Error Handlers

These rules are ALWAYS ACTIVE for all files in `src/core/FileSystemUtils.ts` and `src/cli/handlers.ts` that perform configuration directory resolution, environment variable access, file system validation, and CLI error reporting.

### Rules

- **R-CLI-ERR-001** MUST: Use `console.error` for error logging in configuration and environment management modules (`src/core/FileSystemUtils.ts`, `src/cli/handlers.ts`) to avoid external logging dependencies during bootstrap phases.
- **R-CLI-ERR-002** SHOULD: Prefix error messages with consistent markers (e.g., `[ruler]`, `ERROR_PREFIX` constants) to distinguish error types and improve user experience.
- **R-CLI-ERR-003** SHOULD: Include actionable context in error messages: file paths, directory names, operation types, and suggested remediation steps.
- **R-CLI-ERR-004** MUST: Ensure `console.error` calls occur before any `process.exit()` invocations to guarantee error visibility.
- **R-CLI-ERR-005** MUST NOT: Import external logging libraries (pino, winston, etc.) in configuration and environment management modules.
- **R-CLI-ERR-006** SHOULD: Document the boundary between `console.error` usage (configuration/CLI initialization) and structured logging (application logic) in architecture guidelines.

### Verify

```bash
# Verify console.error usage with contextual prefixes in restricted modules
grep -r 'console\.error' src/core/FileSystemUtils.ts src/cli/handlers.ts | grep -E '(\[ruler\]|ERROR_PREFIX)'

# Verify no external logging library imports in configuration and environment management modules
grep -r 'import.*logger' src/core/FileSystemUtils.ts src/cli/handlers.ts && exit 1 || exit 0

# Verify error logging is functional
node -e "require('./src/core/FileSystemUtils.ts'); require('./src/cli/handlers.ts')" 2>&1 | grep -q 'Error' && echo 'Error logging functional' || echo 'No errors detected'
```

**Accept when:**
- All error logging in `src/core/FileSystemUtils.ts` and `src/cli/handlers.ts` uses `console.error` with contextual prefixes (e.g., `[ruler]`, `ERROR_PREFIX`).
- No external logging library imports are present in configuration and environment management modules.
- Error messages include sufficient context (paths, operation names, directory names) to enable debugging without additional tooling.
- `console.error` calls are positioned before any `process.exit()` invocations to guarantee visibility.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for pull requests modifying configuration and CLI error handling in the specified modules. Violations must be flagged during code review with clear explanations of the architectural rationale.
</enforcement>
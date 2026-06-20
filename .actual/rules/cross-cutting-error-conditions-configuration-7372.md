# Standardize console-based logging for warnings and errors across core modules: Error Conditions Configuration

These rules are ALWAYS ACTIVE for all modules in the Frameworks & Libraries > Libraries & Modules category that perform observability logging operations, specifically: ConfigLoader.ts, FileSystemUtils.ts, FirebenderAgent.ts, and handlers.ts.

### Rules

- **R-CONSOLE-001** MUST: All error conditions in configuration loading, file system operations, and agent initialization MUST be logged using console.error with descriptive context including file paths and error messages.
- **R-CONSOLE-002** MUST: Use console.warn for non-fatal conditions such as missing optional configuration, fallback behaviors, or deprecated usage patterns.
- **R-CONSOLE-003** MUST: Include module-specific prefixes (e.g., '[ruler]', ERROR_PREFIX constant) in log messages to enable filtering and identification in multi-module applications.
- **R-CONSOLE-004** MUST: When logging errors from caught exceptions, include both the operation context and the original error message in the format: `console.error(\`Operation failed: ${err.message}\`)`.
- **R-CONSOLE-005** MUST NOT: Introduce external logging framework dependencies (winston, pino, bunyan, etc.) in core library modules (src/core/, src/agents/).

### Verify

```bash
# Count console.error usage in core modules
grep -r 'console\.error' src/ | grep -E '(ConfigLoader|FileSystemUtils|FirebenderAgent|handlers)' | wc -l

# Count console.warn usage in core modules
grep -r 'console\.warn' src/ | grep -E '(ConfigLoader|FileSystemUtils|FirebenderAgent|handlers)' | wc -l

# Verify no external logging framework imports in core modules
! grep -r 'import.*winston\|import.*pino\|import.*bunyan' src/core/ src/agents/
```

**Accept when:**
- All error conditions in ConfigLoader, FileSystemUtils, FirebenderAgent, and handlers modules are logged using console.error with descriptive context (operation name, affected resource, error message).
- All warning conditions use console.warn with sufficient context to understand the non-fatal issue.
- No external logging framework dependencies (winston, pino, bunyan) are imported in core library modules (src/core/, src/agents/).
- Log messages include module-specific prefixes for filtering and identification.
- Exception handling logs include both operation context and original error message.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All error and warning logging in scope modules MUST conform to console-based patterns with required context. External logging framework imports in core modules MUST be rejected.
</enforcement>
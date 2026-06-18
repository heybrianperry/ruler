# Standardize console-based logging for warnings and errors across core modules: Warning Conditions Non

These rules are ALWAYS ACTIVE for all modules in the Frameworks & Libraries > Libraries & Modules category that perform observability logging operations, specifically: Configuration loading operations (ConfigLoader.ts), File system utilities (FileSystemUtils.ts), Agent initialization and configuration (FirebenderAgent.ts), and CLI command handlers (handlers.ts).

### Rules

- **R-CONSOLE-001** MUST: All warning conditions (non-fatal errors, fallback behaviors) MUST be logged using console.warn with sufficient context to understand the condition.
- **R-CONSOLE-002** MUST: All error conditions that indicate failure or unexpected state MUST be logged using console.error; include the operation name, affected resource (file path, config key), and error message.
- **R-CONSOLE-003** MUST: Include module-specific prefixes (e.g., '[ruler]', ERROR_PREFIX constant) in log messages to enable filtering and identification in multi-module applications.
- **R-CONSOLE-004** MUST: When logging errors from caught exceptions, include both the operation context and the original error message in the format: console.error(`Operation failed: ${err.message}`).
- **R-CONSOLE-005** MUST: No external logging framework dependencies (winston, pino, bunyan, etc.) shall be introduced in core library modules (src/core/, src/agents/).

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
- Module-specific prefixes are consistently applied to log messages for filtering and identification.
- Error logging from caught exceptions includes both operation context and original error message.

<enforcement>
Claude Code MUST NOT skip or defer verification. All five rules must be verified before accepting code changes to ConfigLoader.ts, FileSystemUtils.ts, FirebenderAgent.ts, or handlers.ts. Violations require code review feedback and refactoring requests.
</enforcement>
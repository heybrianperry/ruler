# Standardize console-based logging for warnings and errors across core modules: Log Messages Include

These rules are ALWAYS ACTIVE for all modules in the Frameworks & Libraries > Libraries & Modules category that perform observability logging operations, including ConfigLoader, FileSystemUtils, FirebenderAgent, and CLI command handlers.

### Rules

- **R-LOG-001** SHOULD: Log messages SHOULD include the operation being performed and the specific resource (file path, configuration key) that triggered the condition.
- **R-LOG-002** MUST: Use console.error for all error conditions that indicate failure or unexpected state; include the operation name, affected resource (file path, config key), and error message.
- **R-LOG-003** MUST: Use console.warn for non-fatal conditions such as missing optional configuration, fallback behaviors, or deprecated usage patterns.
- **R-LOG-004** SHOULD: Include module-specific prefixes (e.g., '[ruler]', ERROR_PREFIX constant) in log messages to enable filtering and identification in multi-module applications.
- **R-LOG-005** SHOULD: When logging errors from caught exceptions, include both the operation context and the original error message: console.error(`Operation failed: ${err.message}`).
- **R-LOG-006** MUST: Do not introduce external logging framework dependencies (winston, pino, bunyan) in core library modules (src/core/, src/agents/).

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
- All error conditions in ConfigLoader, FileSystemUtils, FirebenderAgent, and handlers modules are logged using console.error with descriptive context (operation, resource, error message).
- All warning conditions use console.warn with sufficient context to understand the non-fatal issue.
- No external logging framework dependencies (winston, pino, bunyan) are imported in core library modules (src/core/, src/agents/).
- Log messages include module-specific prefixes for filtering and identification.
- Error logging includes both operation context and original error message.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement.
</enforcement>
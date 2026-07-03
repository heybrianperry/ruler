# Standardize console-based logging for warnings and errors across core modules: Core Library Modules

These rules are ALWAYS ACTIVE for all modules in the Frameworks & Libraries > Libraries & Modules category that perform observability logging operations, specifically: Configuration loading operations (ConfigLoader.ts), File system utilities (FileSystemUtils.ts), Agent initialization and configuration (FirebenderAgent.ts), and CLI command handlers (handlers.ts).

### Rules

- **R-CORE-LOG-001** MUST_NOT: Core library modules MUST NOT introduce external logging framework dependencies (e.g., winston, bunyan, pino) that would increase bundle size or create coupling.
- **R-CORE-LOG-002** MUST: Use console.error for all error conditions that indicate failure or unexpected state; include the operation name, affected resource (file path, config key), and error message.
- **R-CORE-LOG-003** MUST: Use console.warn for non-fatal conditions such as missing optional configuration, fallback behaviors, or deprecated usage patterns.
- **R-CORE-LOG-004** SHOULD: Include module-specific prefixes (e.g., '[ruler]', ERROR_PREFIX constant) in log messages to enable filtering and identification in multi-module applications.
- **R-CORE-LOG-005** SHOULD: When logging errors from caught exceptions, include both the operation context and the original error message in the format: console.error(`Operation failed: ${err.message}`).

### Verify

```bash
# Count console.error usage in core modules
grep -r 'console\.error' src/ | grep -E '(ConfigLoader|FileSystemUtils|FirebenderAgent|handlers)' | wc -l

# Count console.warn usage in core modules
grep -r 'console\.warn' src/ | grep -E '(ConfigLoader|FileSystemUtils|FirebenderAgent|handlers)' | wc -l

# Verify no external logging framework dependencies in core modules
! grep -r 'import.*winston\|import.*pino\|import.*bunyan' src/core/ src/agents/
```

**Accept when:**
- All error conditions in ConfigLoader, FileSystemUtils, FirebenderAgent, and handlers modules are logged using console.error with descriptive context (operation name, affected resource, error message).
- All warning conditions use console.warn with sufficient context to understand the non-fatal issue.
- No external logging framework dependencies (winston, pino, bunyan) are imported in core library modules (src/core/, src/agents/).
- Log messages include module-specific prefixes or ERROR_PREFIX constants for filtering and identification.
- Exception handling logs include both operation context and original error message.

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. All error and warning logging in scope modules MUST use console methods exclusively, and MUST NOT introduce external logging dependencies.
</enforcement>
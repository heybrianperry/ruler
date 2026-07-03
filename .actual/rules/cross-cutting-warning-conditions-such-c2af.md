# Standardize Console-Based Logging for Error and Warning Events: Warning Conditions Such

These rules are ALWAYS ACTIVE for all agent implementations and core utility modules that handle error conditions, configuration failures, or file system operations requiring diagnostic output.

### Rules

- **R-LOG-001** MUST: Warning conditions such as recoverable parsing failures or missing optional configuration MUST be logged using console.warn with sufficient detail to diagnose the issue.
- **R-LOG-002** MUST: Use console.error for unrecoverable errors and console.warn for recoverable failures; include the error object as the final argument to preserve stack traces.
- **R-LOG-003** MUST: Prefix log messages with component identifiers (e.g., '[ruler]', '[firebender]') to distinguish output sources in multi-component systems.
- **R-LOG-004** MUST: Include structured context in log messages: file paths, configuration keys, operation names, and relevant state to enable rapid diagnosis.
- **R-LOG-005** MUST NOT: Use console.log in production code paths within integration and file system modules; use console.error or console.warn instead.
- **R-LOG-006** SHOULD: Avoid console logging in loops, recursive functions, or high-frequency event handlers; consider aggregating errors or using conditional logging based on environment variables for verbose output.

### Verify

```bash
# Verify console.error and console.warn usage in scoped modules
grep -r 'console\.error\|console\.warn' src/ --include='*.ts' | grep -E '(FileSystemUtils|Agent)' | wc -l

# Verify no console.log in integration modules
grep -r 'console\.log' src/core/FileSystemUtils.ts src/agents/FirebenderAgent.ts && echo 'FAIL: console.log found in integration modules' || echo 'PASS'

# Count logging-related test coverage
npm test -- --grep 'logging' 2>&1 | grep -E '(error|warn)' | wc -l
```

**Accept when:**
- Verification commands confirm console.error and console.warn usage in FileSystemUtils and agent modules without console.log in production code paths.
- Code review confirms error logs include error objects and sufficient context (file paths, operation names) for diagnosis.
- No console logging appears in performance-critical paths or high-frequency operations identified during profiling.
- Log messages include component prefixes and structured context as specified in R-LOG-003 and R-LOG-004.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules during code review and implementation. All R-LOG rules are mandatory for scoped modules.
</enforcement>
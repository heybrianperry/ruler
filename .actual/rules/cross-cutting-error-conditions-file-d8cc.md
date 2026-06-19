# Standardize Console-Based Logging for Error and Warning Events: Error Conditions File

These rules are ALWAYS ACTIVE for all agent implementations and core utility modules that handle error conditions, configuration failures, or file system operations requiring diagnostic output.

### Rules

- **R-CONSOLE-001** MUST: Error conditions in file system operations, configuration parsing, and directory validation MUST be logged using console.error with descriptive context including the operation attempted and the error object.
- **R-CONSOLE-002** MUST: Use console.error for unrecoverable errors and console.warn for recoverable failures or missing optional configuration; include the error object as the final argument to preserve stack traces.
- **R-CONSOLE-003** MUST: Prefix log messages with component identifiers (e.g., '[ruler]', '[firebender]') to distinguish output sources in multi-component systems.
- **R-CONSOLE-004** MUST: Include structured context in log messages: file paths, configuration keys, operation names, and relevant state to enable rapid diagnosis without additional tooling.
- **R-CONSOLE-005** MUST: Avoid console logging in loops, recursive functions, or high-frequency event handlers; consider aggregating errors or using conditional logging based on environment variables for verbose output.

### Verify

```bash
# Verify console.error and console.warn usage in FileSystemUtils and agent modules
grep -r 'console\.error\|console\.warn' src/ --include='*.ts' | grep -E '(FileSystemUtils|Agent)' | wc -l

# Verify no console.log in production code paths
grep -r 'console\.log' src/core/FileSystemUtils.ts src/agents/FirebenderAgent.ts && echo 'FAIL: console.log found in integration modules' || echo 'PASS'

# Verify logging test coverage
npm test -- --grep 'logging' 2>&1 | grep -E '(error|warn)' | wc -l
```

**Accept when:**
- Verification commands confirm console.error and console.warn usage in FileSystemUtils and agent modules without console.log in production code paths
- Code review confirms error logs include error objects and sufficient context (file paths, operation names) for diagnosis
- No console logging appears in performance-critical paths or high-frequency operations identified during profiling
- Log messages include component prefixes and structured context as specified in R-CONSOLE-003 and R-CONSOLE-004

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules during code review and implementation.
</enforcement>
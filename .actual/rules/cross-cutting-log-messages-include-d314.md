# Standardize Console-Based Logging for Error and Warning Events: Log Messages Include

These rules are ALWAYS ACTIVE for all agent implementations and core utility modules that handle error conditions, configuration failures, or file system operations requiring diagnostic output.

### Rules

- **R-LOG-001** SHOULD: Log messages SHOULD include structured context such as file paths, configuration keys, or operation names to enable rapid diagnosis.
- **R-LOG-002** MUST: Use console.error for unrecoverable errors and console.warn for recoverable failures or missing optional configuration; include the error object as the final argument to preserve stack traces.
- **R-LOG-003** SHOULD: Prefix log messages with component identifiers (e.g., '[ruler]', '[firebender]') to distinguish output sources in multi-component systems.
- **R-LOG-004** MUST NOT: Use console.log in production code paths within integration and file system modules; use console.error or console.warn instead.
- **R-LOG-005** MUST NOT: Add console logging in loops, recursive functions, or high-frequency event handlers; consider aggregating errors or using conditional logging based on environment variables for verbose output.

### Verify

```bash
# Confirm console.error and console.warn usage in FileSystemUtils and agent modules
grep -r 'console\.error\|console\.warn' src/ --include='*.ts' | grep -E '(FileSystemUtils|Agent)' | wc -l

# Verify no console.log in production code paths
grep -r 'console\.log' src/core/FileSystemUtils.ts src/agents/FirebenderAgent.ts && echo 'FAIL: console.log found in integration modules' || echo 'PASS'

# Check test coverage for logging patterns
npm test -- --grep 'logging' 2>&1 | grep -E '(error|warn)' | wc -l
```

**Accept when:**
- Verification commands confirm console.error and console.warn usage in FileSystemUtils and agent modules without console.log in production code paths.
- Code review confirms error logs include error objects and sufficient context (file paths, operation names) for diagnosis.
- No console logging appears in performance-critical paths or high-frequency operations identified during profiling.
- Component identifiers are consistently prefixed in log messages across all scoped modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement.
</enforcement>
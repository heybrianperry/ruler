# Standardize Console-Based Logging for Error and Warning Events: Console Logging Not

These rules are ALWAYS ACTIVE for all agent implementations and core utility modules that handle error conditions, configuration failures, or file system operations requiring diagnostic output.

### Rules

- **R-CONSOLE-001** MUST_NOT: Console logging MUST NOT be used for high-frequency operations or performance-critical paths where output volume would degrade system performance.
- **R-CONSOLE-002** MUST: Use console.error for unrecoverable errors and console.warn for recoverable failures or missing optional configuration; include the error object as the final argument to preserve stack traces.
- **R-CONSOLE-003** SHOULD: Prefix log messages with component identifiers (e.g., '[ruler]', '[firebender]') to distinguish output sources in multi-component systems.
- **R-CONSOLE-004** SHOULD: Include structured context in log messages: file paths, configuration keys, operation names, and relevant state to enable rapid diagnosis without additional tooling.
- **R-CONSOLE-005** MUST_NOT: Avoid console logging in loops, recursive functions, or high-frequency event handlers; consider aggregating errors or using conditional logging based on environment variables for verbose output.
- **R-CONSOLE-006** MUST_NOT: Do not use console.log in production code paths for integration and file system modules; use console.error or console.warn instead.

### Verify

```bash
# Verify console.error and console.warn usage in FileSystemUtils and agent modules
grep -r 'console\.error\|console\.warn' src/ --include='*.ts' | grep -E '(FileSystemUtils|Agent)' | wc -l

# Verify no console.log in integration modules
grep -r 'console\.log' src/core/FileSystemUtils.ts src/agents/FirebenderAgent.ts && echo 'FAIL: console.log found in integration modules' || echo 'PASS'

# Count error/warn logging in tests
npm test -- --grep 'logging' 2>&1 | grep -E '(error|warn)' | wc -l
```

**Accept when:**
- Verification commands confirm console.error and console.warn usage in FileSystemUtils and agent modules without console.log in production code paths.
- Code review confirms error logs include error objects and sufficient context (file paths, operation names) for diagnosis.
- No console logging appears in performance-critical paths or high-frequency operations identified during profiling.
- Log messages include component prefixes and structured context for rapid diagnosis.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules during code review. All violations of R-CONSOLE-001, R-CONSOLE-005, and R-CONSOLE-006 MUST be flagged and corrected before acceptance.
</enforcement>
# Standardize Console-Based Logging for Error and Warning Events: Error Logs Include

These rules are ALWAYS ACTIVE for all agent implementations and core utility modules that handle error conditions, configuration failures, or file system operations requiring diagnostic output.

### Rules

- **R-CONSOLE-001** SHOULD: Error logs SHOULD include the original error object or message to preserve stack traces and diagnostic information.

### Verify

```bash
# Verify console.error and console.warn usage in FileSystemUtils and agent modules
grep -r 'console\.error\|console\.warn' src/ --include='*.ts' | grep -E '(FileSystemUtils|Agent)' | wc -l

# Verify no console.log in production code paths
grep -r 'console\.log' src/core/FileSystemUtils.ts src/agents/FirebenderAgent.ts && echo 'FAIL: console.log found in integration modules' || echo 'PASS'

# Count error/warn logging in test suite
npm test -- --grep 'logging' 2>&1 | grep -E '(error|warn)' | wc -l
```

**Accept when:**
- Verification commands confirm console.error and console.warn usage in FileSystemUtils and agent modules without console.log in production code paths
- Code review confirms error logs include error objects and sufficient context (file paths, operation names) for diagnosis
- No console logging appears in performance-critical paths or high-frequency operations identified during profiling

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must pass before accepting changes to scoped modules.
</enforcement>
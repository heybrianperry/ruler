# Subprocess Execution via child_process: Subprocess Executions Specify Execution Timeout Limits

These rules are ALWAYS ACTIVE for all core utility modules performing system-level inspection and external process execution, and test harness initialization scripts orchestrating environment setup and external processes.

### Rules

- **R-SUB-001** SHOULD: Subprocess executions specify execution timeout limits and cancellation signal handlers to prevent stalled process leaks.
- **R-SUB-002** MUST: Submissions violating exception constraints (EXC-20-001) or failing to provide distinct array elements for command arguments rather than interpolated shell strings must be rejected.

### Verify

```bash
# Discover the project test execution script from the repository manifest and run test suites covering subprocess execution.
# Inspect the project configuration to identify and run the static analysis and linting scripts against module boundaries.
```

**Accept when:**
- All test suites verifying subprocess invocation and error handling pass with exit code zero.
- Static verification confirms all child_process invocations provide explicit error handling and timeout boundaries.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
# Subprocess Execution via child_process: Invocations Child Process Routines Handle Exit

These rules are ALWAYS ACTIVE for core utility modules performing system-level inspection and external process execution, and test harness initialization scripts orchestrating environment setup and external processes.

### Rules

- **R-CP-001** MUST: All invocations of child_process routines MUST handle exit codes, standard error output, and process lifecycle failure events explicitly.
- **R-CP-002** MUST: Receive command arguments as distinct array elements rather than interpolated shell strings.
- **R-CP-003** MUST: Ensure standard error and standard output streams are drained or piped properly to avoid buffer exhaustion deadlocks.

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
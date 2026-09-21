# Subprocess Execution via child_process: Modules Utilize Synchronous Process Execution Methods

These rules are ALWAYS ACTIVE for core utility modules performing system-level inspection/external process execution and test harness initialization scripts orchestrating environment setup.

### Rules

- **R-PROC-001** MAY: Modules MAY utilize synchronous process execution methods during initialization phases where asynchronous event loops are not yet established.
- **R-PROC-002** MANDATORY: Prohibit raw shell string concatenation and mandate argument array passing for subprocess execution to prevent command injection vulnerabilities.
- **R-PROC-003** MANDATORY: Enforce explicit timeout configurations and signal-based cancellation on all process invocations to prevent indefinite hanging.
- **R-PROC-004** MANDATORY: Ensure standard error and standard output streams are drained or piped properly to avoid buffer exhaustion deadlocks.

### Verify

```bash
# Discover the project test execution script from the repository manifest and run test suites covering subprocess execution.
# Inspect the project configuration to identify and run the static analysis and linting scripts against module boundaries.
```

**Accept when:**
- All test suites verifying subprocess invocation and error handling pass with exit code zero.
- Static verification confirms all child_process invocations provide explicit error handling and timeout boundaries.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated continuous integration pipeline running test suites and static analysis checks, and peer code review.
</enforcement>
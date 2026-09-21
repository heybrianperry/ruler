# Subprocess Execution via child_process: Child Process Module Serve Standard Mechanism

These rules are ALWAYS ACTIVE for core utility modules performing system-level inspection/external process execution and test harness initialization scripts orchestrating environment setup.

### Rules

- **R-CP-001** MUST: The `child_process` module MUST serve as the standard mechanism for delegating execution tasks to operating system subprocesses across core utilities and environment setup.
- **R-CP-002** MUST: Subprocess invocation calls MUST receive command arguments as distinct array elements rather than interpolated shell strings to prevent command injection vulnerabilities.
- **R-CP-003** MUST: Standard error and standard output streams MUST be drained or piped properly to avoid buffer exhaustion deadlocks.
- **R-CP-004** MUST: Explicit timeout configurations and signal-based cancellation MUST be enforced on all process invocations to prevent indefinite hanging.
- **R-CP-005** MAY: Synchronous subprocess execution is permitted strictly during initial environment bootstrap before the asynchronous loop begins (EXC-20-001).

### Verify

```bash
# Discover the project test execution script from the repository manifest and run test suites covering subprocess execution.
# Inspect the project configuration to identify and run the static analysis and linting scripts against module boundaries.
# (Exact commands depend on project repository setup - run package test and lint scripts)
```

**Accept when:**
- All test suites verifying subprocess invocation and error handling pass with exit code zero.
- Static verification confirms all child_process invocations provide explicit error handling, argument separation, and timeout boundaries.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated continuous integration pipelines and peer code reviews enforce these requirements, resulting in build failures or review rejections for unhandled process executions, raw string concatenations, or missing timeout parameters.
</enforcement>
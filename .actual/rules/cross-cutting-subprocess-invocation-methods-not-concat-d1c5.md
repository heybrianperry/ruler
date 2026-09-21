# Subprocess Execution via child_process: Subprocess Invocation Methods Not Concatenate Unvalidated

These rules are ALWAYS ACTIVE for all core utility modules performing system-level inspection/external process execution, and test harness initialization scripts orchestrating environment setup and external processes.

### Rules

- **R-SUB-001** MUST_NOT: Subprocess invocation methods MUST NOT concatenate unvalidated or untrusted external input strings into shell command execution strings.

### Verify

```bash
# Discover the project test execution script from the repository manifest and run test suites covering subprocess execution.
# Inspect the project configuration to identify and run the static analysis and linting scripts against module boundaries.
```

**Accept when:**
- All test suites verifying subprocess invocation and error handling pass with exit code zero.
- Static verification confirms all child_process invocations provide explicit error handling and timeout boundaries.

<enforcement>
Claude Code MUST NOT skip or defer verification. All subprocess invocation calls must receive command arguments as distinct array elements rather than interpolated shell strings.
</enforcement>
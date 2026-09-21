# Subprocess Execution via child_process: Consumers Inspect Repository Lock Resolution Artifact

These rules are ALWAYS ACTIVE for core utility modules performing system-level inspection and external process execution, and test harness initialization scripts orchestrating environment setup and external processes.

### Rules

- **R-SPE-001** MUST: Consumers MUST inspect the repository lock or resolution artifact to determine the exact platform runtime and dependency versions before invoking version-dependent process execution behavior.

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
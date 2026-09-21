# Subprocess Execution via child_process: Core Utility Implementations Isolate Child Process

These rules are ALWAYS ACTIVE for all core utility modules performing system-level inspection, external process execution, and test harness initialization scripts.

### Rules

- **R-SUB-001** SHOULD: Core utility implementations SHOULD isolate child_process invocations behind domain-specific abstraction wrappers to permit test substitution.

### Verify

```bash
# Discover and run test suites covering subprocess execution from project manifest
# Run static analysis and linting scripts against module boundaries
```

**Accept when:**
- All test suites verifying subprocess invocation and error handling pass with exit code zero.
- Static verification confirms all child_process invocations provide explicit error handling and timeout boundaries.

<enforcement>
Claude Code MUST NOT skip or defer verification. All subprocess invocation calls must receive command arguments as distinct array elements rather than interpolated shell strings, and standard error and standard output streams must be drained or piped properly to avoid buffer exhaustion deadlocks.
</enforcement>
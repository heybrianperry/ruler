# Adoption of child_process Module for Subprocess Execution: Before Implementing Functionality That Relies Versioned

These rules are ALWAYS ACTIVE for components requiring interaction with local operating system binaries, host command execution, test harness, and environment initialization routines needing sub-process orchestration.

### Rules

- **R-CP-001** MUST: Before implementing functionality that relies on versioned runtime modules or dependencies, the consumer MUST inspect the repository lock artifact to determine the authoritative resolved runtime version and verify API compatibility against official documentation.
- **R-CP-002** MUST: Enforce argument array parameterization via spawn or execFile rather than shell string interpolation, verified through code review.
- **R-CP-003** MUST: Use streaming interfaces with explicit buffer limits or consume streams incrementally to prevent unbounded buffer consumption.
- **R-CP-004** SHOULD: Encapsulate recurring subprocess invocations within dedicated utility wrapper functions to centralize stream parsing and error propagation.
- **R-CP-005** SHOULD: Ensure all child processes are gracefully terminated upon parent process teardown to prevent orphaned zombie processes.

### Verify

```bash
# Discover and run the repository test suite verifying that utility functions and test setup scripts execute subprocess calls without error.
# Discover and execute the repository static analysis and security scanning tasks to verify secure invocation patterns of process execution methods.
```

**Accept when:**
- All tests verifying subprocess operations pass with zero unhandled stream errors or non-zero exit code failures.
- Static analysis checks confirm no unsanitized command string concatenations are present in subprocess execution calls.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated continuous integration test suites, static analysis linting rules, and peer code review.
</enforcement>
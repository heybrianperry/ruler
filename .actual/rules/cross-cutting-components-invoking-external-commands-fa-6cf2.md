# Adoption of child_process Module for Subprocess Execution: Components Invoking External Commands Favor Asynchronous

These rules are ALWAYS ACTIVE for components requiring interaction with local operating system binaries, host command execution, test harness, and environment initialization routines needing sub-process orchestration.

### Rules

- **R-CHIL-001** SHOULD: Components invoking external commands favor asynchronous process invocation methods such as spawn or execFile over synchronous counterparts to prevent blocking the event loop.

### Verify

```bash
# Discover and run the repository test suite verifying that utility functions and test setup scripts execute subprocess calls without error.
# Discover and execute the repository static analysis and security scanning tasks to verify secure invocation patterns of process execution methods.
```

**Accept when:**
- All tests verifying subprocess operations pass with zero unhandled stream errors or non-zero exit code failures.
- Static analysis checks confirm no unsanitized command string concatenations are present in subprocess execution calls.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
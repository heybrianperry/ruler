# Adoption of child_process Module for Subprocess Execution: Application Use Core Child Process Module

These rules are ALWAYS ACTIVE for all files interacting with local operating system binaries, host command execution, and test harness or environment initialization routines needing sub-process orchestration.

### Rules

- **R-CP-001** MUST: The application MUST use the core child_process module for spawning and orchestrating external operating system processes and external command execution.

### Verify

```bash
# Discover and run the repository test suite verifying subprocess calls
# Discover and execute static analysis and security scanning tasks for secure invocation patterns
```

**Accept when:**
- All tests verifying subprocess operations pass with zero unhandled stream errors or non-zero exit code failures.
- Static analysis checks confirm no unsanitized command string concatenations are present in subprocess execution calls.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated continuous integration test suites, static analysis linting rules, and peer code reviews enforce these requirements.
</enforcement>
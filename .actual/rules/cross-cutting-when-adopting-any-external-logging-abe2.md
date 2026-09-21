# Console API Logging: When Adopting Any External Logging Dependency

These rules are ALWAYS ACTIVE for all application modules emitting diagnostic, warning, verbose, or error output to standard terminal streams, command handling, filesystem operations, and agent execution layers requiring runtime operational status reporting.

### Rules

- **R-LOG-001** MUST: When adopting any external logging dependency or versioned telemetry library, the consumer MUST inspect the repository lock or resolution artifact to determine the exact resolved version before implementation.

### Verify

```bash
discover_and_run_linter
discover_and_run_test_suite
```

**Accept when:**
- Static analysis passes with zero unhandled or non-standard console errors across all source files.
- All automated unit and integration test suites pass without unexpected stderr emissions.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
# Console API Logging: Modules Requiring Diagnostic Output Restrict Direct

These rules are ALWAYS ACTIVE for all application modules emitting diagnostic, warning, verbose, or error output to standard terminal streams, including command handling, filesystem operations, and agent execution layers requiring runtime operational status reporting.

### Rules

- **R-LOG-001** MUST: Modules requiring diagnostic output MUST restrict direct console invocations to standard runtime error and warning methods with standardized namespace prefix identifiers.

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
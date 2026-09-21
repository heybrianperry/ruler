# Console API Logging: Diagnostic Messages Emitted Through Console Error

These rules are ALWAYS ACTIVE for all application modules emitting diagnostic, warning, verbose, or error output to standard terminal streams, including command handling, filesystem operations, and agent execution layers requiring runtime operational status reporting.

### Rules

- **R-LOG-001** MUST: Diagnostic messages emitted through console error and warning methods MUST include structured context and sanitized error descriptions to prevent secret leakage.

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
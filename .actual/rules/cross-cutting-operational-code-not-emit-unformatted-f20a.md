# Console API Logging: Operational Code Not Emit Unformatted Unstructured

These rules are ALWAYS ACTIVE for all application modules emitting diagnostic, warning, verbose, or error output to standard terminal streams, including command handling, filesystem operations, and agent execution layers.

### Rules

- **R-LOG-001** MUST_NOT: Operational code MUST NOT emit unformatted, unstructured raw objects to standard console streams where downstream consumers expect clean command output.

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
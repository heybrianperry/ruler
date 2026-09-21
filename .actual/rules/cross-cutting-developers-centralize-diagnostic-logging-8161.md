# Console API Logging: Developers Centralize Diagnostic Logging Wrappers Within

These rules are ALWAYS ACTIVE for all application modules emitting diagnostic, warning, verbose, or error output to standard terminal streams, including command handling, filesystem operations, and agent execution layers requiring runtime operational status reporting.

### Rules

- **R-LOG-001** SHOULD: Developers centralize diagnostic logging wrappers within shared utility constants rather than scattering raw console invocations across domain logic.
- **R-LOG-002** MANDATORY: Format diagnostic errors using centralized error formatting helpers before passing messages to console error streams.
- **R-LOG-003** MANDATORY: Ensure standard output and standard error separation is maintained so diagnostic logs do not pollute command standard output.
- **R-LOG-004** MANDATORY: Execute lock-version grounding in order (find manifest, identify build tool, inspect lock/resolution, look up official docs, confirm APIs, re-run per dependency at point of use) before writing code that uses a versioned library.

### Verify

```bash
# Discover and run linter
discover_and_run_linter

# Discover and run test suite
discover_and_run_test_suite
```

**Accept when:**
- Static analysis passes with zero unhandled or non-standard console errors across all source files.
- All automated unit and integration test suites pass without unexpected stderr emissions.

<enforcement>
Verification is mandatory. Claude Code MUST NOT skip or defer verification.
</enforcement>
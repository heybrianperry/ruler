# Console API Logging: New Components Route Diagnostic Verbose Execution

These rules are ALWAYS ACTIVE for all application modules emitting diagnostic, warning, verbose, or error output to standard terminal streams, including command handling, filesystem operations, and agent execution layers requiring runtime operational status reporting.

### Rules

- **R-LOG-001** MAY: New components route diagnostic and verbose execution messages through namespaced verbose output channels that respect runtime verbosity flags.
- **R-LOG-002** MANDATORY: Format diagnostic errors using centralized error formatting helpers before passing messages to console error streams, and ensure standard output and standard error separation is maintained so diagnostic logs do not pollute command standard output.
- **R-LOG-003** MANDATORY: Enforce error formatting utilities that sanitize input arguments and strip sensitive environment variables before logging.
- **R-LOG-004** MANDATORY: Define centralized prefix constants and verify compliance via static analysis and automated code review to ensure consistent error prefixing.
- **R-LOG-005** MANDATORY (LOCK-VERSION GROUNDING): Before writing code that uses a versioned library, execute in order: (1) Find the dependency manifest in the repo declaring ranges; (2) Identify the build tool from the manifest; (3) Inspect the repository lock or resolution artifact to determine the exact resolved version; (4) Look up official documentation, changelog, or public API reference for that exact version; (5) Confirm every API, class, or function you will call exists in that exact version's documentation; (6) Re-run steps 3-5 per dependency at point of use.

### Verify

```bash
# Discover and run linter and test suite according to project configuration
discover_and_run_linter
discover_and_run_test_suite
```

**Accept when:**
- Static analysis passes with zero unhandled or non-standard console errors across all source files.
- All automated unit and integration test suites pass without unexpected stderr emissions.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests containing unapproved raw console calls or non-compliant output formatting will fail automated review checks. Exceptions require an architectural review request with explicit sign-off from the technical lead and documentation in the corresponding module specification.
</enforcement>
# ConfigLoader Module Adoption: Core Engine Subsystems Retrieve Runtime Options

These rules are ALWAYS ACTIVE for core engine subsystem modules requiring operational parameters or runtime configuration settings.

### Rules

- **R-CFG-001** MUST: Core engine subsystems MUST retrieve all runtime options and operational parameters exclusively through ConfigLoader.

### Verify

```bash
# Discover the test runner defined in the project configuration and execute the core test suite to verify configuration loading behavior.
# Discover the static analysis command from the repository scripts and verify that core modules have no unauthorized configuration imports.
# Discover the build script from the repository manifest and execute a clean build to confirm interface compatibility with ConfigLoader.
```

**Accept when:**
- Core engine subsystems successfully initialize their runtime options exclusively via ConfigLoader.
- Direct parsing of filesystem configuration artifacts or raw environment variables is absent from core operational modules.
- Repository test and verification suites pass with centralized configuration resolution in place.

<enforcement>
Claude Code MUST NOT skip or defer verification. All changes touching core subsystem configuration access are verified by static analysis, dependency graph inspection, and peer review.
</enforcement>
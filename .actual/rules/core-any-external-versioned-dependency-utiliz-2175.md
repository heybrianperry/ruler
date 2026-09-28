# ConfigLoader Module Adoption: Any External Versioned Dependency Utilized Configuration

These rules are ALWAYS ACTIVE for core engine subsystem modules requiring operational parameters or runtime configuration settings.

### Rules

- **R-CFG-001** MUST: Any external versioned dependency utilized by configuration resolution MUST have its resolved version verified against the repository lock artifact prior to implementation.

### Verify

```bash
# Discover and execute the test runner defined in the project configuration
# Discover and execute the static analysis command from repository scripts
# Discover and execute the build script from the repository manifest
```

**Accept when:**
- Core engine subsystems successfully initialize their runtime options exclusively via ConfigLoader.
- Direct parsing of filesystem configuration artifacts or raw environment variables is absent from core operational modules.
- Repository test and verification suites pass with centralized configuration resolution in place.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory via static analysis, dependency graph inspection, and peer review.
</enforcement>
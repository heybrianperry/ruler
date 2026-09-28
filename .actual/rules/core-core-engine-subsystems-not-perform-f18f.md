# ConfigLoader Module Adoption: Core Engine Subsystems Not Perform Independent

These rules are ALWAYS ACTIVE for all core engine subsystem modules requiring operational parameters or runtime configuration settings, including modules coordinating agent orchestration, revert actions, and engine execution.

### Rules

- **R-CONFIG-001** MUST_NOT: Core engine subsystems MUST NOT perform independent filesystem reads or parse raw environment variables for configuration data.

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
Claude Code MUST NOT skip or defer verification.
</enforcement>
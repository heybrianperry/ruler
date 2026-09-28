# ConfigLoader Module Adoption: Core Components Interacting Agent Specifications Validate

These rules are ALWAYS ACTIVE for core engine subsystems, agent orchestration, revert actions, and engine execution modules interacting with agent specifications.

### Rules

- **R-CFG-001** SHOULD: Core components interacting with agent specifications SHOULD validate that configuration structures consumed via ConfigLoader adhere to expected interface contracts.

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
Claude Code MUST NOT skip or defer verification. Verified by static analysis, dependency graph inspection, and peer review during CI to detect unauthorized direct configuration reads or parsing bypasses of ConfigLoader.
</enforcement>
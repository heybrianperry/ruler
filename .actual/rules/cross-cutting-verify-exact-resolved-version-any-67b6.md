# Adoption of Core Utility Libraries and Shared Type Definitions: Verify Exact Resolved Version Any External

These rules are ALWAYS ACTIVE for code that interacts with external dependencies, file systems, YAML configurations, or shared type definitions.

### Rules

- **R-VER-001** MUST: verify the exact resolved version of any external dependency by inspecting the project's lock file before implementing or updating code that relies on it.

### Verify

```bash
# Discover the project's build configuration and run the type-checking command.
# Discover the project's test runner and execute all unit and integration tests.
# Discover the project's dependency management tool and inspect the lock file for resolved versions of core utility libraries.
```

**Accept when:**
- Type-checking completes without errors, indicating correct usage of shared types.
- All tests pass, confirming the functionality relying on core utilities.
- The lock file clearly specifies the versions of adopted core utility libraries.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
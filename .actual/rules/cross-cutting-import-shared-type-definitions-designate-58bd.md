# Adoption of Core Utility Libraries and Shared Type Definitions: Import Shared Type Definitions Designated Internal

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-ADR-001** MUST: MUST import shared type definitions from the designated internal module to ensure type consistency across the codebase.

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
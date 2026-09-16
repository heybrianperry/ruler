# Adoption of Core Utility Libraries and Shared Type Definitions: Prefer Existing Utility Functions Adopted Core

These rules are ALWAYS ACTIVE for all code within the project, particularly modules responsible for configuration loading, file system interactions, subagent utilities, and shared data structures.

### Rules

- **R-ADOPT-001** SHOULD: Prefer existing utility functions from adopted core libraries over custom implementations for common tasks.
- **R-ADOPT-002** MUST: Before writing code that uses a versioned library, execute in order: Find the dependency manifest in the repo; Identify the build tool from the manifest; Inspect the repository lock or resolution artifact to determine the exact resolved version; Look up the official documentation, changelog, or public API reference for that exact version; Confirm every API, class, or function you will call exists in that exact version's documentation before using it; For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- **R-ADOPT-003** SHOULD: Add new utility functions to existing core utility modules if they align with their purpose, rather than creating new, isolated utility files.
- **R-ADOPT-004** MUST: Define all new shared types in the designated shared types module.

### Verify

```bash
# Discover the project's build configuration and run the type-checking command.
$(discover_build_config_and_typecheck)

# Discover the project's test runner and execute all unit and integration tests.
$(discover_test_runner_and_run_tests)

# Discover the project's dependency management tool and inspect the lock file for resolved versions of core utility libraries.
$(discover_dependency_tool_and_inspect_lockfile)
```

**Accept when:**
- Type-checking completes without errors, indicating correct usage of shared types.
- All tests pass, confirming the functionality relying on core utilities.
- The lock file clearly specifies the versions of adopted core utility libraries.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
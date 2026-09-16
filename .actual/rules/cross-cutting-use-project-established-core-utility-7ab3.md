# Adoption of Core Utility Libraries and Shared Type Definitions: Use Project Established Core Utility Libraries

These rules are ALWAYS ACTIVE for all code that interacts with file systems, parses YAML, or uses shared type definitions.

### Rules

- **R-CORE-001** MUST: Use the project's established core utility libraries for file path manipulation, asynchronous file system operations, and YAML configuration parsing.

### Verify

```bash
# Discover the project's build configuration and run the type-checking command.
# (e.g., `npm run typecheck`, `tsc --noEmit`, `mvn compile`)

# Discover the project's test runner and execute all unit and integration tests.
# (e.g., `npm test`, `pytest`, `mvn test`)

# Discover the project's dependency management tool and inspect the lock file for resolved versions of core utility libraries.
# (e.g., `npm ls --json`, `yarn why`, `pip freeze`, `grep -r "core-utility-lib" yarn.lock`, `cat package-lock.json | jq '.dependencies."your-core-lib".version'`)
```

**Accept when:**
- Type-checking completes without errors, indicating correct usage of shared types.
- All tests pass, confirming the functionality relying on core utilities.
- The lock file clearly specifies the versions of adopted core utility libraries.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
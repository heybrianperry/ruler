# Adoption of Node.js Core Modules and @iarna/toml for System Operations: Before Implementing Any Code That Uses

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-ADR-001** MUST: Before implementing any code that uses a versioned library, the consumer MUST discover the ecosystem's lock file, identify the build tool, inspect the repository lock or resolution artifact to determine the exact resolved version, and consult version-specific documentation to confirm API existence and behavior.

### Verify

```bash
# Discover the project's static analysis configuration and execute it to identify direct imports of Node.js core modules and `@iarna/toml`.
# Locate the project's test suite and run all relevant tests to ensure system interactions and TOML parsing behave as expected.
# Examine the project's dependency manifest and lock file to confirm the presence and resolved version of `@iarna/toml`.
```

**Accept when:**
- Static analysis reports no unauthorized direct system interaction outside of established patterns.
- All tests related to file system operations, path handling, process execution, and TOML parsing pass successfully.
- The `@iarna/toml` library is present in the dependency graph at a resolved version.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
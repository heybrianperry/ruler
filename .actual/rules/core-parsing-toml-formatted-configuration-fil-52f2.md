# Adoption of Node.js Core Modules and @iarna/toml for System Operations: Parsing Toml Formatted Configuration Files Handled

These rules are ALWAYS ACTIVE for core utility modules, engine components, and any code requiring direct system interaction, path manipulation, process execution, or TOML parsing.

### Rules

- **R-ADOPTION-001** MUST: Parsing of TOML formatted configuration files MUST be handled by the `@iarna/toml` library.

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
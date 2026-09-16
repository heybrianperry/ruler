# Adoption of Zod and Iarna/TOML for Configuration Management and Validation: Discover Ecosystem Lock File Resolve Exact

These rules are ALWAYS ACTIVE for modules responsible for loading, parsing, or validating application configuration, and core utility modules interacting with the file system or operating system.

### Rules

- **R-LOCK-001** MUST: MUST discover the ecosystem's lock file and resolve the exact locked version of all third-party dependencies before implementation.

### Verify

```bash
# Inspect the project's dependency manifest to confirm the presence of the specified libraries.
# Examine configuration loading modules to verify `zod` schema definitions and `@iarna/toml` usage.
# Run the project's test suite to ensure configuration loading and validation behave as expected.
```

**Accept when:**
- All configuration loading paths successfully validate against their respective `zod` schemas.
- TOML configuration files are correctly parsed by the designated library.
- File system and OS interactions function as intended without errors.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
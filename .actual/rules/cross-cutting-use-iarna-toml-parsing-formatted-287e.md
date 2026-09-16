# Adoption of Zod and Iarna/TOML for Configuration Management and Validation: Use Iarna Toml Parsing Formatted Configuration

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-TOML-001** MUST: Use `@iarna/toml` for parsing TOML-formatted configuration files.

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
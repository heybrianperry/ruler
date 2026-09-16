# Adoption of Zod and Iarna/TOML for Configuration Management and Validation: Use Zod Defining Validating Application Configuration

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-ZOD-001** MUST: MUST use `zod` for defining and validating all application configuration schemas.

### Verify

```bash
# Inspect the project's dependency manifest to confirm the presence of the specified libraries.
# Example: grep -r "zod" package.json yarn.lock
# Example: grep -r "@iarna/toml" package.json yarn.lock

# Examine configuration loading modules to verify zod schema definitions and @iarna/toml usage.
# Example: grep -r "z.object" src/config/
# Example: grep -r "@iarna/toml" src/config/

# Run the project's test suite to ensure configuration loading and validation behave as expected.
# Example: npm test
# Example: yarn test
```

**Accept when:**
- All configuration loading paths successfully validate against their respective `zod` schemas.
- TOML configuration files are correctly parsed by the designated library.
- File system and OS interactions function as intended without errors.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
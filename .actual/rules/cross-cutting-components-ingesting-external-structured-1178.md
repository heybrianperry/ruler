# Zod Schema Validation: Components Ingesting External Structured Configuration Validate

These rules are ALWAYS ACTIVE for components ingesting external structured configuration, parsing persistent configuration data, and constructing internal runtime models from external serialized payloads.

### Rules

- **R-ZOD-001** MUST: Components ingesting external structured configuration MUST validate parsed data through Zod schema objects that enforce explicit field types, optionality, and strict key constraints.

### Verify

```bash
# Discover and execute the project unit test suite for configuration and schema validation modules
# Run the repository static analysis and type checking scripts to verify that schema types align with declared contracts
```

**Accept when:**
- The configuration validation test suite passes with complete verification of valid, invalid, and legacy input shapes.
- All external configuration loading paths reject invalid inputs with structured error notifications during automated test verification.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
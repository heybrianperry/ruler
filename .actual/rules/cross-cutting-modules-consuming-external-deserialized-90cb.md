# Zod Schema Validation: Modules Consuming External Deserialized Input Not

These rules are ALWAYS ACTIVE for modules consuming external deserialized input, configuration loading, and external agent integrations.

### Rules

- **R-ZOD-001** MUST_NOT: Modules consuming external deserialized input MUST NOT cast raw parsed structures directly to internal types without prior schema execution and validation.

### Verify

```bash
# Discover and execute the project unit test suite for configuration and schema validation modules.
# Run the repository static analysis and type checking scripts to verify that schema types align with declared contracts.
```

**Accept when:**
- The configuration validation test suite passes with complete verification of valid, invalid, and legacy input shapes.
- All external configuration loading paths reject invalid inputs with structured error notifications during automated test verification.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests introducing unvalidated deserialization on external inputs must be blocked.
</enforcement>
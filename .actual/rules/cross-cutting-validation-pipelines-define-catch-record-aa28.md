# Zod Schema Validation: Validation Pipelines Define Catch Record Schemas

These rules are ALWAYS ACTIVE for parsing and loading of persistent configuration data from environment paths and storage locations, and the construction of internal runtime models from external serialized payloads.

### Rules

- **R-ZOD-001** MAY: Validation pipelines MAY define catch-all record schemas for arbitrary key-value mapping blocks when configuration keys cannot be predetermined.

### Verify

```bash
# Discover and execute the project unit test suite for configuration and schema validation modules
# Run the repository static analysis and type checking scripts to verify that schema types align with declared contracts
```

**Accept when:**
- The configuration validation test suite passes with complete verification of valid, invalid, and legacy input shapes.
- All external configuration loading paths reject invalid inputs with structured error notifications during automated test verification.

<enforcement>
Claude Code MUST NOT skip or defer verification. All external ingestion paths must declare explicit schemas, and schema parsing failures at runtime must log actionable diagnostic messages and abort initialization.
</enforcement>
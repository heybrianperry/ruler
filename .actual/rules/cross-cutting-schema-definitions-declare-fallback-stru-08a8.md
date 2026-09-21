# Zod Schema Validation: Schema Definitions Declare Fallback Structures Deprecation

These rules are ALWAYS ACTIVE for parsing and loading of persistent configuration data from environment paths, storage locations, and external agent integrations constructing internal runtime definitions.

### Rules

- **R-ZOD-001** SHOULD: Schema definitions SHOULD declare fallback structures and deprecation handling using optional schema fields to preserve backward compatibility during format transitions.

### Verify

```bash
# Discover and execute the project unit test suite for configuration and schema validation modules
# Run repository static analysis and type checking scripts to verify schema types align with contracts
```

**Accept when:**
- The configuration validation test suite passes with complete verification of valid, invalid, and legacy input shapes.
- All external configuration loading paths reject invalid inputs with structured error notifications during automated test verification.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests introducing unvalidated deserialization on external inputs must be blocked.
</enforcement>
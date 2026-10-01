# Zod Schema Validation: Schema Definitions Apply Strict Mode Constraints

These rules are ALWAYS ACTIVE for ingestion of external configuration files, structured payloads, and persistent state representations mapping to internal domain interfaces.

### Rules

- **R-ZOD-001** SHOULD: Schema definitions apply strict mode constraints to reject unrecognized configuration keys unless an explicit catchall schema is required for dynamic attributes.

### Verify

```bash
# Discover and execute repository verification scripts covering boundary parsing and schema validation
# Discover and execute static analysis and type-checking scripts to verify schema and type alignment
```

**Accept when:**
- All configuration parsing test suites execute successfully and validate both valid and invalid schema inputs.
- Static analysis checks pass with zero type errors across all schema definition modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated continuous integration test suites and peer code reviews verify schema validation on test fixtures.
</enforcement>
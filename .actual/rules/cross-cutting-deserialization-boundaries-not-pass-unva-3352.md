# Zod Schema Validation: Deserialization Boundaries Not Pass Unvalidated Objects

These rules are ALWAYS ACTIVE for external configuration ingestion, structured payloads at application entry points, and persistent state deserialization mapping to internal domain interfaces.

### Rules

- **R-ZOD-001** MUST_NOT: Deserialization boundaries MUST NOT pass unvalidated objects directly into core domain structures without prior schema validation.

### Verify

```bash
# Discover and execute the repository verification scripts for boundary parsing and schema validation
# Discover and execute static analysis and type-checking scripts
```

**Accept when:**
- All configuration parsing test suites execute successfully and validate both valid and invalid schema inputs.
- Static analysis checks pass with zero type errors across all schema definition modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated continuous integration test suites and peer code reviews verify schema validation on test fixtures and newly introduced deserialization points.
</enforcement>
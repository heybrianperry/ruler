# Zod Schema Validation: Input Validation Schemas Define Optional Fallback

These rules are ALWAYS ACTIVE for ingestion of external configuration files, structured payloads, and persistent state representations mapping to internal domain interfaces.

### Rules

- **R-ZOD-001** MAY: Input validation schemas MAY define optional fallback keys to support backward compatibility when migrating deprecated schema properties.

### Verify

```bash
# Discover and execute the repository verification scripts to run test suites covering boundary parsing and schema validation.
# Discover and execute the repository static analysis and type-checking scripts to verify schema and type alignment.
```

**Accept when:**
- All configuration parsing test suites execute successfully and validate both valid and invalid schema inputs.
- Static analysis checks pass with zero type errors across all schema definition modules.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
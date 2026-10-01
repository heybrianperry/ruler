# Zod Schema Validation: Engineering Team Use Zod Schema Definitions

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-ZOD-001** MUST: The engineering team MUST use Zod schema definitions with explicit shape definitions and strict boundary checks to validate all external configuration data ingested from persistent storage or environment sources.

### Verify

```bash
# Discover and execute repository verification scripts covering boundary parsing and schema validation
# Discover and execute repository static analysis and type-checking scripts
```

**Accept when:**
- All configuration parsing test suites execute successfully and validate both valid and invalid schema inputs.
- Static analysis checks pass with zero type errors across all schema definition modules.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
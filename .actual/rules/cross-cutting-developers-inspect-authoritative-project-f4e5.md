# Zod Schema Validation: Developers Inspect Authoritative Project Dependency Lock

These rules are ALWAYS ACTIVE for configuration loading, external agent integrations, and external input artifact parsing.

### Rules

- **R-ZOD-001** MUST: Developers MUST inspect the authoritative project dependency lock artifact to determine the exact resolved version of Zod before utilizing its schema validation APIs.

### Verify

```bash
# Discover and execute the project unit test suite for configuration and schema validation modules
# Run repository static analysis and type checking scripts to verify schema types align with declared contracts
```

**Accept when:**
- The configuration validation test suite passes with complete verification of valid, invalid, and legacy input shapes.
- All external configuration loading paths reject invalid inputs with structured error notifications during automated test verification.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
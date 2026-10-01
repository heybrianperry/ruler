# Zod Schema Validation: Before Writing Code That Uses Versioned

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-ZOD-001** MUST: Before writing code that uses the versioned schema validation dependency, developers MUST inspect the repository lock or resolution artifact to determine the exact resolved dependency version and verify API compatibility against official documentation.

### Verify

```bash
# Discover and execute repository verification scripts for boundary parsing and schema validation
# Discover and execute static analysis and type-checking scripts
```

**Accept when:**
- All configuration parsing test suites execute successfully and validate both valid and invalid schema inputs.
- Static analysis checks pass with zero type errors across all schema definition modules.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
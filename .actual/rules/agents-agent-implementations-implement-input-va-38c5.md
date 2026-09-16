# Adoption of Core Agent Modules and Input Validation: Agent Implementations Implement Input Validation Structured

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-AGENT-INPUT-VALIDATION-001** MUST: Agent implementations MUST implement input validation for all structured data consumed, utilizing appropriate parsing mechanisms.

### Verify

```bash
Discover and execute the project's static analysis tools.
Discover and execute the project's unit and integration tests for agent modules.
Discover and execute the project's dependency audit tools.
```

**Accept when:**
- Static analysis reports no violations of module import patterns for agent-related files.
- All agent-related unit and integration tests pass successfully.
- Dependency audit confirms consistent versioning of core modules and their dependencies.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
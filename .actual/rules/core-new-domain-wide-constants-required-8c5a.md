# Adoption of Internal Constants Module: New Domain Wide Constants Required Multiple

These rules are ALWAYS ACTIVE for core execution processors, agent selection services, and shared utility modules within the core domain.

### Rules

- **R-DOM-001** SHOULD: New domain-wide constants required by multiple processors or utilities SHOULD be declared within the centralized constants module rather than within processor-specific scopes.
- **R-EXC-001** MAY: Exempt components that require runtime-dynamic configuration values loaded from external dynamic sources that cannot be represented as static constants (Exception: EXC-20-001).

### Verify

```bash
# Discover and run the project static analysis and linting scripts
# Discover and execute the project unit and integration test suites
```

**Accept when:**
- Static analysis checks pass with zero violations regarding unauthorized duplicated constants or magic literals in core processors.
- All automated test suites for core processors and agent selection modules pass successfully.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
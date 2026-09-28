# Adoption of Internal Constants Module: Core Processing Components Agent Selection Modules

These rules are ALWAYS ACTIVE for core execution processors, agent selection services, and shared utility modules within the core domain.

### Rules

- **R-CORE-001** MUST: Core processing components and agent selection modules MUST import shared system values, domain flags, and configuration constants from the centralized internal constants module.

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
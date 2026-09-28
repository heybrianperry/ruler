# Adoption of Internal Constants Module: Consumers Discover Project Dependency Manifest Lock

These rules are ALWAYS ACTIVE for core execution processors, agent selection services, shared utility modules within the core domain, and all components that define or consume cross-cutting domain constants and shared operational parameters.

### Rules

- **R-ICM-001** MUST: Consumers MUST discover the project dependency manifest and lock artifact to resolve all dependency versions before implementing or updating module integrations.

### Verify

```bash
# Discover and run the project static analysis and linting scripts to verify compliance with centralized constants import rules.
# Discover and execute the project unit and integration test suites to confirm that dependent processors operate correctly with centralized constant values.
```

**Accept when:**
- Static analysis checks pass with zero violations regarding unauthorized duplicated constants or magic literals in core processors.
- All automated test suites for core processors and agent selection modules pass successfully.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
# Adoption of Internal Constants Module: Modules Outside Core Domain Boundary Not

These rules are ALWAYS ACTIVE for core execution processors, agent selection services, and shared utility modules within the core domain boundary.

### Rules

- **R-MOD-001** SHOULD_NOT: Modules outside the core domain boundary SHOULD NOT export or re-export internal core constants directly to avoid leaking internal implementation boundaries.

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
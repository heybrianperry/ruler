# Adoption of Internal Constants Module: Core Components Not Define Local Duplicate

These rules are ALWAYS ACTIVE for all core domain processors, agent selection services, and shared utility modules within the core domain.

### Rules

- **R-CONST-001** MUST_NOT: Core components MUST NOT define local duplicate definitions of shared constants or introduce hardcoded literal values that represent cross-cutting domain parameters.

### Verify

```bash
# Discover and run the project static analysis and linting scripts
# Discover and execute the project unit and integration test suites
```

**Accept when:**
- Static analysis checks pass with zero violations regarding unauthorized duplicated constants or magic literals in core processors.
- All automated test suites for core processors and agent selection modules pass successfully.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification by automated static analysis checks and peer code review is mandatory.
</enforcement>
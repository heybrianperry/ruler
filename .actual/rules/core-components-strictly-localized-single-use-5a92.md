# Adoption of Internal Constants Module: Components Strictly Localized Single Use Internal

These rules are ALWAYS ACTIVE for core domain processors, agent selection routines, and processing utilities requiring access to shared configuration values and domain parameters.

### Rules

- **R-CONST-001** MAY: Components with strictly localized, single-use internal configurations MAY maintain private scoped definitions when those values are not referenced across module boundaries.

### Verify

```bash
# Discover and run project static analysis and linting scripts to verify compliance with centralized constants import rules
# Discover and execute project unit and integration test suites to confirm dependent processors operate correctly with centralized constant values
```

**Accept when:**
- Static analysis checks pass with zero violations regarding unauthorized duplicated constants or magic literals in core processors.
- All automated test suites for core processors and agent selection modules pass successfully.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated static analysis and peer code review enforce compliance; pull requests introducing duplicate constant definitions or bypasses are blocked.
</enforcement>
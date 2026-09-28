# Adoption of types Internal Module for Shared Data Contracts: Before Introducing Updating Any Third Party

These rules are ALWAYS ACTIVE for all core domain processors, utility modules, configuration management, external settings adaptation layers, and cross-subsystem communication interfaces.

### Rules

- **R-ADR-001** MUST: Before introducing or updating any third-party dependency utilized by contract definitions or serialization routines, the consumer MUST inspect the repository lock artifact to determine the exact resolved dependency version and verify API compatibility against official documentation.

### Verify

```bash
# Discover and run the project static analysis and type verification script
# Discover and run the module dependency cycle analysis script
# Discover and run the project automated test suite
```

**Accept when:**
- Static type verification passes with zero errors across all core utilities, processors, and configuration modules.
- Dependency graph verification reports zero circular dependency cycles involving the centralized types module.
- All unit and integration test suites succeed with all assertions passing against shared contract definitions.

<enforcement>
Claude Code MUST NOT skip or defer verification. All pull requests failing static type checks or introducing circular dependencies are automatically blocked from merging.
</enforcement>
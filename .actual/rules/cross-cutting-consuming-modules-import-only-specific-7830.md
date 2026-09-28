# Adoption of types Internal Module for Shared Data Contracts: Consuming Modules Import Only Specific Types

These rules are ALWAYS ACTIVE for all core domain processors, utility modules, configuration management layers, and cross-subsystem communication interfaces.

### Rules

- **R-TYP-001** SHOULD: Consuming modules import only the specific types and interfaces required for their operational scope rather than importing broad namespace aggregates.

### Verify

```bash
# Discover and execute the project static analysis and type verification script from the repository manifest
# Discover and execute the module dependency cycle analysis script from the build configuration
# Discover and run the project automated test suite from the repository manifest
```

**Accept when:**
- Static type verification passes with zero errors across all core utilities, processors, and configuration modules.
- Dependency graph verification reports zero circular dependency cycles involving the centralized types module.
- All unit and integration test suites succeed with all assertions passing against shared contract definitions.

<enforcement>
Claude Code MUST NOT skip or defer verification. All pull requests failing static type checks or introducing circular dependencies are automatically blocked from merging.
</enforcement>
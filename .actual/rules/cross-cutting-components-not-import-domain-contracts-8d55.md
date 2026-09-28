# Adoption of types Internal Module for Shared Data Contracts: Components Not Import Domain Contracts Directly

These rules are ALWAYS ACTIVE for all core domain processors, utility modules, configuration management layers, and cross-subsystem communication interfaces.

### Rules

- **R-TYPES-001** MUST_NOT: Components MUST NOT import domain contracts directly from peer implementation modules across subsystem boundaries, routing all inter-domain type requirements through the centralized types module to avoid circular dependency chains.

### Verify

```bash
# Discover and execute the project static analysis and type verification script
# Discover and execute the module dependency cycle analysis script
# Discover and run the project automated test suite
```

**Accept when:**
- Static type verification passes with zero errors across all core utilities, processors, and configuration modules.
- Dependency graph verification reports zero circular dependency cycles involving the centralized types module.
- All unit and integration test suites succeed with all assertions passing against shared contract definitions.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory for all pull requests and code changes affecting domain contracts and cross-subsystem dependencies.
</enforcement>
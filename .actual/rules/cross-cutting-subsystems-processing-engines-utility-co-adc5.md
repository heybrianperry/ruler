# Adoption of types Internal Module for Shared Data Contracts: Subsystems Processing Engines Utility Components Configuration

These rules are ALWAYS ACTIVE for core domain logic, processors, configuration validators, integration layers, subsystems, processing engines, utility components, and configuration adapters.

### Rules

- **R-TYPES-001** MUST: Subsystems, processing engines, utility components, and configuration adapters MUST import all shared domain contracts, interfaces, and common data definitions from the centralized types module rather than declaring local contract duplicates.

### Verify

```bash
# Discover the project static analysis and type verification script from the repository manifest and execute it
# Discover and execute the module dependency cycle analysis script from the build configuration
# Discover and run the project automated test suite from the repository manifest
```

**Accept when:**
- Static type verification passes with zero errors across all core utilities, processors, and configuration modules.
- Dependency graph verification reports zero circular dependency cycles involving the centralized types module.
- All unit and integration test suites succeed with all assertions passing against shared contract definitions.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory via automated type checking, linting pipelines, architecture dependency boundary checks, and peer reviews.
</enforcement>
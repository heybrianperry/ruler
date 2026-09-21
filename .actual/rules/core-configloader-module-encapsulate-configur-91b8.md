# ConfigLoader Module Adoption for Core Configuration Management: Configloader Module Encapsulate Configuration Parsing Schema

These rules are ALWAYS ACTIVE for all core engine modules and orchestrators requiring runtime configuration settings.

### Rules

- **R-CFG-001** SHOULD: The ConfigLoader module SHOULD encapsulate configuration parsing, schema validation, and fallback resolution for all core consumers.

### Verify

```bash
# Discover and run the repository test runner script for the core subsystem test suite
# Discover and run the repository static analysis script across core modules
```

**Accept when:**
- All core engine subsystem tests pass without configuration loading errors.
- Static verification confirms all core engine modules source runtime configuration exclusively through the ConfigLoader module.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated continuous integration pipeline executes static analysis and test validation suites.
</enforcement>
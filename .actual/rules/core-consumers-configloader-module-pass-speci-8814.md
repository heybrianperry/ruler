# ConfigLoader Module Adoption for Core Configuration Management: Consumers Configloader Module Pass Specific Override

These rules are ALWAYS ACTIVE for core engine modules and orchestrators requiring runtime configuration settings.

### Rules

- **R-CFG-001** MAY: Consumers of the ConfigLoader module MAY pass module-specific override parameters when initializing or querying configuration if permitted by the module interface.

### Verify

```bash
# Discover the repository test runner script from the project configuration and execute the core subsystem test suite.
# Discover the repository static analysis script from the project configuration and execute static verification across core modules.
```

**Accept when:**
- All core engine subsystem tests pass without configuration loading errors.
- Static verification confirms all core engine modules source runtime configuration exclusively through the ConfigLoader module.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory.
</enforcement>
# ConfigLoader Module Adoption for Core Configuration Management: Core Engine Components Requiring Runtime Configuration

These rules are ALWAYS ACTIVE for core engine modules and orchestrators requiring runtime configuration settings.

### Rules

- **R-CORE-001** MUST: Core engine components requiring runtime configuration MUST retrieve operational settings through the ConfigLoader module.
- **R-CORE-002** MUST: Core modules requiring configuration must import ConfigLoader rather than directly inspecting configuration sources.

### Verify

```bash
# Discover the repository test runner script from the project configuration and execute the core subsystem test suite.
# Discover the repository static analysis script from the project configuration and execute static verification across core modules.
```

**Accept when:**
- All core engine subsystem tests pass without configuration loading errors.
- Static verification confirms all core engine modules source runtime configuration exclusively through the ConfigLoader module.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory and enforced by automated CI pipelines and peer code review.
</enforcement>
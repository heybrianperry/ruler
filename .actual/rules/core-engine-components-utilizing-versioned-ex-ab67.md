# ConfigLoader Module Adoption for Core Configuration Management: Engine Components Utilizing Versioned External Dependencies

These rules are ALWAYS ACTIVE for all core engine modules, orchestrators, and components requiring runtime configuration settings.

### Rules

- **R-CFG-001** MUST: Engine components utilizing versioned external dependencies MUST discover the project resolution artifact and verify the exact locked dependency version before invoking external APIs.
- **R-CFG-002** MUST: Core engine modules requiring configuration must import ConfigLoader rather than directly inspecting configuration sources.

### Verify

```bash
# Discover and execute the repository test runner script from project configuration
# Discover and execute the repository static analysis script from project configuration
```

**Accept when:**
- All core engine subsystem tests pass without configuration loading errors.
- Static verification confirms all core engine modules source runtime configuration exclusively through the ConfigLoader module.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory.
</enforcement>
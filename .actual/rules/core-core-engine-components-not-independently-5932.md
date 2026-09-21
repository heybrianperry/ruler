# ConfigLoader Module Adoption for Core Configuration Management: Core Engine Components Not Independently Parse

These rules are ALWAYS ACTIVE for core engine modules, orchestrators, and runtime configuration consumers.

### Rules

- **R-CORE-001** MUST_NOT: Core engine components MUST NOT independently parse configuration files or bypass the ConfigLoader module to read raw configuration values.

### Verify

```bash
# Discover and execute the repository static analysis and core subsystem test suite according to project configuration
```

**Accept when:**
- All core engine subsystem tests pass without configuration loading errors.
- Static verification confirms all core engine modules source runtime configuration exclusively through the ConfigLoader module.

<enforcement>
Claude Code MUST NOT skip or defer verification. Continuous integration test failures and code reviews will block bypasses.
</enforcement>
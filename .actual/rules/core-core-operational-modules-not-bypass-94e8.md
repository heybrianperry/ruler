# ConfigLoader Module Adoption for Core Configuration Management: Core Operational Modules Not Bypass Configloader

These rules are ALWAYS ACTIVE for all core operational components and engine subsystems requiring runtime or operational configuration.

### Rules

- **R-CFG-001** MUST_NOT: Core operational modules MUST_NOT bypass ConfigLoader to access raw configuration files, disk resources, or environment properties directly.

### Verify

```bash
# Discover the project static analysis and linting scripts from the repository build manifest and execute them across core subsystem source modules.
# Discover the test runner command from the repository build manifest and execute all unit and integration test suites covering core modules.
```

**Accept when:**
- Static analysis confirms zero direct environment or file system configuration reads within core operational modules outside ConfigLoader.
- All unit and integration tests covering core modules pass with configuration provided through ConfigLoader.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
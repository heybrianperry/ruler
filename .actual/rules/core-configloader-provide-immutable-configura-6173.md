# ConfigLoader Module Adoption for Core Configuration Management: Configloader Provide Immutable Configuration Value Access

These rules are ALWAYS ACTIVE for all core operational components and engine subsystems requiring runtime or operational configuration.

### Rules

- **R-CFG-001** SHOULD: ConfigLoader SHOULD provide immutable configuration value access to consumer modules to prevent runtime mutations.

### Verify

```bash
# Discover project static analysis, linting, and test runner scripts from the repository build manifest
# and execute them across core subsystem source modules and test suites.
```

**Accept when:**
- Static analysis confirms zero direct environment or file system configuration reads within core operational modules outside ConfigLoader.
- All unit and integration tests covering core modules pass with configuration provided through ConfigLoader.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
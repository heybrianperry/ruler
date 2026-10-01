# ConfigLoader Module Adoption for Core Configuration Management: Core Modules Requiring Configuration Updates Subscribe

These rules are ALWAYS ACTIVE for all core operational components and engine subsystems requiring runtime or operational configuration.

### Rules

- **R-CFG-001** SHOULD: Core modules requiring configuration updates SHOULD subscribe to or query ConfigLoader through defined module interfaces rather than maintaining local configuration copies.

### Verify

```bash
# Discover and execute project static analysis and linting scripts from the repository build manifest across core subsystem source modules.
# Discover and execute the test runner command from the repository build manifest covering all unit and integration test suites for core modules.
```

**Accept when:**
- Static analysis confirms zero direct environment or file system configuration reads within core operational modules outside ConfigLoader.
- All unit and integration tests covering core modules pass with configuration provided through ConfigLoader.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated static analysis rules preventing direct environment variable access in core modules and mandatory peer review.
</enforcement>
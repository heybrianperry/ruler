# Constants Module Adoption for Core Configuration and Domain Values: Core Operational Modules Import System Configuration

These rules are ALWAYS ACTIVE for core operational modules, processing units, selection components, and filesystem utilities requiring access to shared system keys, operational defaults, or cross-cutting configuration parameters.

### Rules

- **R-CONST-001** MUST: Core operational modules MUST import system configuration keys, default values, and operational limits from the centralized constants module rather than declaring local literal values.

### Verify

```bash
# Discover and run the project static analysis and type verification script
# Discover and run the project test suite
```

**Accept when:**
- Static analysis and type checking pass with zero errors across all core modules importing from the constants module.
- All unit and integration test suites pass successfully, confirming consistent behavior across core processing workflows.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
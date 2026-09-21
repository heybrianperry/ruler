# Constants Module Adoption for Core Configuration and Domain Values: Core Processing Modules Not Duplicate Literal

These rules are ALWAYS ACTIVE for core subsystem modules and utilities requiring access to shared system keys, operational defaults, or cross-cutting configuration parameters, including components coordinating agent selection, subagent processing, skills processing, and filesystem operations.

### Rules

- **R-CONST-001** MUST_NOT: Core processing modules MUST NOT duplicate literal values for system-wide configuration keys, environment variable references, or default operational thresholds across module boundaries.

### Verify

```bash
# Discover and run the project static analysis and type verification script
# Discover and run the project test suite to verify constant definitions
```

**Accept when:**
- Static analysis and type checking pass with zero errors across all core modules importing from the constants module.
- All unit and integration test suites pass successfully, confirming consistent behavior across core processing workflows.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory via static analysis checks and peer code review.
</enforcement>
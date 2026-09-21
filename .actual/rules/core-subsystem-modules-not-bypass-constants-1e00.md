# Constants Module Adoption for Core Configuration and Domain Values: Subsystem Modules Not Bypass Constants Module

These rules are ALWAYS ACTIVE for all core subsystem modules, processing units, selection components, skills processing, and filesystem utilities requiring access to shared system keys, operational defaults, or cross-cutting configuration parameters.

### Rules

- **R-CONST-001** SHOULD_NOT: Subsystem modules SHOULD NOT bypass the constants module by hardcoding fallback values directly in processing routines.

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
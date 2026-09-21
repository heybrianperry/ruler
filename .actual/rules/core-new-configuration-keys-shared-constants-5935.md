# Constants Module Adoption for Core Configuration and Domain Values: New Configuration Keys Shared Constants Defined

These rules are ALWAYS ACTIVE for core subsystem modules, utilities, and components coordinating agent selection, subagent processing, skills processing, and filesystem operations.

### Rules

- **R-CONST-001** SHOULD: New configuration keys and shared constants SHOULD be defined with explicit types and exported immutably from the constants module.
- **R-CONST-002** MANDATORY: Ensure all constants exported from the central module are defined as read-only or immutable structures to prevent runtime mutation.
- **R-CONST-003** MANDATORY: Group constants logically by operational concern to maintain discoverability as the system evolves.

### Verify

```bash
# Discover and run project static analysis and type verification script
# Discover and run the project test suite
```

**Accept when:**
- Static analysis and type checking pass with zero errors across all core modules importing from the constants module.
- All unit and integration test suites pass successfully, confirming consistent behavior across core processing workflows.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated static analysis and peer code reviews enforce compliance.
</enforcement>
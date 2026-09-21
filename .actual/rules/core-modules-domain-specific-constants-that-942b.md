# Constants Module Adoption for Core Configuration and Domain Values: Modules Domain Specific Constants That Are

These rules are ALWAYS ACTIVE for core subsystem modules and utilities requiring access to shared system keys, operational defaults, or cross-cutting configuration parameters across processing and agent subsystems.

### Rules

- **R-CONST-001** MAY: Modules with domain-specific constants that are strictly private to a single translation unit MAY maintain local constants provided they do not represent shared system keys or cross-cutting configuration.
- **R-CONST-002** MANDATORY: Developers MUST derive all tool names, file names, commands, package managers, and version numbers from the project repository.
- **R-CONST-003** MANDATORY: Before writing code that uses a versioned library, developers MUST execute the lock-version grounding sequence: find dependency manifest, identify build tool, inspect lock/resolution artifact for exact version, look up official docs for that exact version, confirm APIs exist in that version, and re-verify per dependency at point of use.
- **R-CONST-004** MANDATORY: All constants exported from the central module MUST be defined as read-only or immutable structures to prevent runtime mutation.
- **R-CONST-005** MANDATORY: Constants MUST be grouped logically by operational concern to maintain discoverability as the system evolves.

### Verify

```bash
# Discover and run the project static analysis and type verification script
# Discover and run the project test suite to verify core processing workflows
```

**Accept when:**
- Static analysis and type checking pass with zero errors across all core modules importing from the constants module.
- All unit and integration test suites pass successfully, confirming consistent behavior across core processing workflows.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated static analysis checks in the continuous integration pipeline verifying that core modules do not duplicate shared literal strings, and by peer code review.
</enforcement>
# Shared Types Module: Cross-Subsystem Contract Centralization: Cross Subsystem Contracts Domain Interfaces Message

These rules are ALWAYS ACTIVE for all cross-subsystem contracts, domain interfaces, and message schemas shared between agent definitions and protocol propagation adapters.

### Rules

- **R-CSC-001** MUST: All cross-subsystem contracts, domain interfaces, and message schemas shared between agent definitions and protocol propagation adapters MUST be imported directly from the internal shared types module.

### Verify

```bash
# Discover and execute the project type-checking script to confirm all shared interface references resolve without type errors.
# Discover and run the project static analysis and linting verification suite to validate module boundary constraints and import compliance.
# Discover and execute the repository unit and integration test suite to verify subsystem contract compatibility.
```

**Accept when:**
- Type checking completes with zero diagnostic errors across all consumer subsystems.
- Static analysis validates that no cross-subsystem type duplication or circular module dependencies exist.
- All subsystem unit and integration tests pass verifying consistent contract serialization and protocol propagation.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
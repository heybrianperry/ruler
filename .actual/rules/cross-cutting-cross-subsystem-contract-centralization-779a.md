# Shared Types Module: Cross-Subsystem Contract Centralization: Before Introducing Modifying External Dependencies That

These rules are ALWAYS ACTIVE for interface definitions, domain contracts, and data transfer schemas shared across distinct subsystem boundaries, as well as agent abstractions and protocol propagation modules interacting with common data models.

### Rules

- **R-SC-001** MUST: Before introducing or modifying external dependencies that feed shared type definitions, the consumer MUST inspect the repository lock artifact to determine the exact resolved dependency version and verify API compatibility.

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
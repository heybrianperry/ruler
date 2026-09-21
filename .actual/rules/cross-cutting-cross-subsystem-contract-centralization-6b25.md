# Shared Types Module: Cross-Subsystem Contract Centralization: Internal Subsystems Define Module Local Helper

These rules are ALWAYS ACTIVE for interface definitions, domain contracts, and data transfer schemas shared across distinct subsystem boundaries, agent abstractions, and protocol propagation modules.

### Rules

- **R-SS-001** MAY: Internal subsystems MAY define module-local helper types within their own boundaries provided those types do not cross subsystem interfaces.

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
Claude Code MUST NOT skip or defer verification. Automated continuous integration pipeline running type verification and lint analysis verifies every pull request. Violations (duplicate cross-subsystem interface definitions or circular type imports) are blocked from merging.
</enforcement>
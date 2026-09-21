# Shared Types Module: Cross-Subsystem Contract Centralization: Shared Types Module Not Import Concrete

These rules are ALWAYS ACTIVE for all interface definitions, domain contracts, and data transfer schemas shared across distinct subsystem boundaries, agent abstractions, and protocol propagation modules.

### Rules

- **R-STM-001** MUST_NOT: The shared types module MUST NOT import concrete runtime implementations, services, or adapters from consuming subsystems, maintaining strict directional decoupling.

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
Claude Code MUST NOT skip or defer verification. Verification is mandatory.
</enforcement>
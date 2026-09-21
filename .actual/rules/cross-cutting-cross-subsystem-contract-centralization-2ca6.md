# Shared Types Module: Cross-Subsystem Contract Centralization: Subsystem Implementations Not Redefine Duplicate Maintain

These rules are ALWAYS ACTIVE for all interface definitions, domain contracts, and data transfer schemas shared across distinct subsystem boundaries, including agent abstractions and protocol propagation modules interacting with common data models.

### Rules

- **R-SUBSYS-001** MUST_NOT: Subsystem implementations MUST NOT redefine, duplicate, or maintain shadow copies of interfaces and data structures established within the shared types module.
- **R-SUBSYS-002** MANDATORY: The consumer MUST discover and execute the project type-checking script to confirm all shared interface references resolve without type errors.
- **R-SUBSYS-003** MANDATORY: The consumer MUST discover and run the project static analysis and linting verification suite to validate module boundary constraints and import compliance.
- **R-SUBSYS-004** MANDATORY: The consumer MUST discover and execute the repository unit and integration test suite to verify subsystem contract compatibility.

### Verify

```bash
# Discover and execute project type-checking, linting, and test suites per repository conventions
```

**Accept when:**
- Type checking completes with zero diagnostic errors across all consumer subsystems.
- Static analysis validates that no cross-subsystem type duplication or circular module dependencies exist.
- All subsystem unit and integration tests pass verifying consistent contract serialization and protocol propagation.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated CI pipeline running type verification and lint analysis on every pull request, as well as architectural peer review.
</enforcement>
# Shared Types Module: Cross-Subsystem Contract Centralization: Consuming Modules Restrict Imports Shared Types

These rules are ALWAYS ACTIVE for interface definitions, domain contracts, and data transfer schemas shared across distinct subsystem boundaries, as well as agent abstractions and protocol propagation modules interacting with common data models.

### Rules

- **R-STC-001** SHOULD: Consuming modules restrict imports from the shared types module exclusively to type definitions and interface declarations, avoiding execution-time side effects.

### Verify

```bash
# Discover and execute the project type-checking script
# Discover and run the project static analysis and linting verification suite
# Discover and execute the repository unit and integration test suite
```

**Accept when:**
- Type checking completes with zero diagnostic errors across all consumer subsystems.
- Static analysis validates that no cross-subsystem type duplication or circular module dependencies exist.
- All subsystem unit and integration tests pass verifying consistent contract serialization and protocol propagation.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated continuous integration pipeline runs type verification and lint analysis on every pull request.
</enforcement>
# AbstractAgent Base Module Adoption: Concrete Agent Classes Not Reimplement Core

These rules are ALWAYS ACTIVE for authoring new agent integrations, concrete assistant implementations, and refactoring existing agent modules within the agent subsystem.

### Rules

- **R-AGNT-001** MUST_NOT: Concrete agent classes MUST NOT reimplement core filesystem resolution or execution mechanics already provided by the AbstractAgent base module.

### Verify

```bash
# Discover the project test execution script from repository manifests and run all unit tests targeting the agent subsystem.
# Discover the static analysis and type checking verification scripts from repository manifests and run them across all agent modules.
```

**Accept when:**
- All concrete agent modules compile successfully and inherit from the AbstractAgent base module.
- Project test suites and static analysis verification pass with zero errors across all agent implementations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated static analysis, type verification executed during continuous integration, and mandatory architectural peer review.
</enforcement>
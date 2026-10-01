# AbstractAgent Base Module Adoption: New Agent Implementations Registered Exported Through

These rules are ALWAYS ACTIVE for authoring new agent integrations, concrete assistant implementations, and refactoring existing agent modules within the agent subsystem.

### Rules

- **R-AGT-001** SHOULD: New agent implementations SHOULD be registered and exported through the centralized agent module barrier to provide a unified consumer interface.
- **R-AGT-002** MANDATORY: Subclasses extending AbstractAgent must invoke base constructor methods and satisfy required abstract lifecycle hooks.
- **R-AGT-003** MANDATORY: Expose concrete implementations via the subsystem index module to maintain clean module boundaries for external consumers.

### Verify

```bash
# Discover the project test execution script from repository manifests and run all unit tests targeting the agent subsystem.
# Discover the static analysis and type checking verification scripts from repository manifests and run them across all agent modules.
```

**Accept when:**
- All concrete agent modules compile successfully and inherit from the AbstractAgent base module.
- Project test suites and static analysis verification pass with zero errors across all agent implementations.

<enforcement>
Verified by automated static analysis and type verification executed during continuous integration, and mandatory architectural peer review of pull requests introducing or modifying agent modules. Pull requests containing agent implementations that bypass the AbstractAgent base module will be blocked until refactored.
</enforcement>
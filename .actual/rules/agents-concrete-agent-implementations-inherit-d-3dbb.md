# AbstractAgent Base Module Adoption: Concrete Agent Implementations Inherit Directly Internal

These rules are ALWAYS ACTIVE for authoring new agent integrations, concrete assistant implementations, and refactoring existing agent modules within the agent subsystem.

### Rules

- **R-AGT-001** MUST: All concrete agent implementations MUST inherit directly from the internal AbstractAgent base module to ensure uniform execution lifecycle and interface conformance.
- **R-AGT-002** MUST: Subclasses extending AbstractAgent must invoke base constructor methods and satisfy required abstract lifecycle hooks.
- **R-AGT-003** MUST: Expose concrete implementations via the subsystem index module to maintain clean module boundaries for external consumers.
- **R-AGT-004** MUST: Restrict AbstractAgent to universal lifecycle mechanics and delegate provider-specific handling to derived implementations to avoid base module bloat.
- **R-AGT-005** MUST: Enforce regression test suites covering all concrete derived agent implementations prior to base class modifications.

### Verify

```bash
# Discover the project test execution script from repository manifests and run all unit tests targeting the agent subsystem.
# Discover the static analysis and type checking verification scripts from repository manifests and run them across all agent modules.
```

**Accept when:**
- All concrete agent modules compile successfully and inherit from the AbstractAgent base module.
- Project test suites and static analysis verification pass with zero errors across all agent implementations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests containing agent implementations that bypass the AbstractAgent base module will be blocked until refactored.
</enforcement>
# AbstractAgent Base Module Adoption: Agent Modules Extending Abstractagent Implement Common

These rules are ALWAYS ACTIVE for authoring new agent integrations, refactoring existing agent modules, and concrete assistant implementations within the agent subsystem.

### Rules

- **R-AGT-001** MUST: All agent modules extending AbstractAgent MUST implement the common agent interface contract exposed by the agent subsystem.
- **R-AGT-002** MUST: Subclasses extending AbstractAgent must invoke base constructor methods and satisfy required abstract lifecycle hooks.
- **R-AGT-003** MUST: Expose concrete implementations via the subsystem index module to maintain clean module boundaries for external consumers.

### Verify

```bash
# Discover the project test execution script from repository manifests and run all unit tests targeting the agent subsystem.
# Discover the static analysis and type checking verification scripts from repository manifests and run them across all agent modules.
```

**Accept when:**
- All concrete agent modules compile successfully and inherit from the AbstractAgent base module.
- Project test suites and static analysis verification pass with zero errors across all agent implementations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory via automated static analysis, type verification, and architectural peer review.
</enforcement>
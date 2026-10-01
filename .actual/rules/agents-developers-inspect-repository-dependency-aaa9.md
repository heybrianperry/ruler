# AbstractAgent Base Module Adoption: Developers Inspect Repository Dependency Resolution Artifact

These rules are ALWAYS ACTIVE for authoring new agent integrations, concrete assistant implementations, and refactoring existing agent modules within the agent subsystem.

### Rules

- **R-AGENTS-001** MUST: Developers MUST inspect the repository dependency resolution artifact to resolve exact pinned versions for all associated runtime and utility dependencies before authoring or modifying agent modules.

### Verify

```bash
# Discover the project test execution script from repository manifests and run all unit tests targeting the agent subsystem.
# Discover the static analysis and type checking verification scripts from repository manifests and run them across all agent modules.
```

**Accept when:**
- All concrete agent modules compile successfully and inherit from the AbstractAgent base module.
- Project test suites and static analysis verification pass with zero errors across all agent implementations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated static analysis, type verification, and mandatory architectural peer reviews will block pull requests that bypass the AbstractAgent base module or fail to inspect dependency resolution artifacts.
</enforcement>
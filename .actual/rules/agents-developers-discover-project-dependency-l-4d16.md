# Adoption of AgentsMdAgent as Base Implementation for Agent Modules: Developers Discover Project Dependency Lock Artifact

These rules are ALWAYS ACTIVE for all specialized agent adapter and integration modules within the agent subsystem.

### Rules

- **R-AG-001** MUST: Developers MUST discover the project dependency lock artifact and resolve all dependency versions before implementing modules that depend on AgentsMdAgent.
- **R-AG-002** MUST: When authoring a new agent module, extend AgentsMdAgent and implement specialized hooks rather than reimplementing execution workflows.
- **R-AG-003** MUST: Verify that internal module imports maintain subsystem boundaries and adhere to shared agent interface expectations.

### Verify

```bash
# Discover and run the project static analysis suite to verify that agent modules import AgentsMdAgent.
# Discover and run the project test suite to validate that all agent implementations satisfy regression and integration tests.
```

**Accept when:**
- All agent integration modules resolve and import AgentsMdAgent without static analysis errors.
- The project test runner executes and passes all test suites covering the agent subsystem.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated static analysis checks and architecture reviews will block pull requests lacking base module inheritance or violating dependency lock guidelines.
</enforcement>
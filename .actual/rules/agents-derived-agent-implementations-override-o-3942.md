# Adoption of AgentsMdAgent as Base Implementation for Agent Modules: Derived Agent Implementations Override Only Specialized

These rules are ALWAYS ACTIVE for all specialized agent adapter and integration modules within the agent subsystem.

### Rules

- **R-AGT-001** SHOULD: Derived agent implementations SHOULD override only specialized execution hooks while delegating base workflows to AgentsMdAgent.
- **R-AGT-002** MANDATORY: When authoring a new agent module, extend AgentsMdAgent and implement specialized hooks rather than reimplementing execution workflows.
- **R-AGT-003** MANDATORY: Verify that internal module imports maintain subsystem boundaries and adhere to shared agent interface expectations.
- **R-AGT-004** MANDATORY (LOCK-VERSION GROUNDING): Before writing code that uses a versioned library, execute in order: (1) Find dependency manifest declaring ranges, (2) Identify build tool, (3) Inspect repo lock/resolution artifact for exact version, (4) Look up official docs/changelog for that exact version, (5) Confirm every API/class/function exists in that version's docs, (6) Re-run steps 3-5 per dependency at point of use for version-sensitive behavior.

### Verify

```bash
# Discover and run the project static analysis suite to verify that agent modules import AgentsMdAgent.
# Discover and run the project test suite to validate that all agent implementations satisfy regression and integration tests.
# (Note: Specific commands must be derived from the project repository as per project discovery policy)
```

**Accept when:**
- All agent integration modules resolve and import AgentsMdAgent without static analysis errors.
- The project test runner executes and passes all test suites covering the agent subsystem.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated static analysis checks and architecture/code reviews for pull requests modifying or adding agent modules. Pull requests lacking base module inheritance are blocked and require refactoring to extend AgentsMdAgent unless an architecture review exception is obtained.
</enforcement>
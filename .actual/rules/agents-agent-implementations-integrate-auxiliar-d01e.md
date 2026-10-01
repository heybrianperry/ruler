# Adoption of AgentsMdAgent as Base Implementation for Agent Modules: Agent Implementations Integrate Auxiliary Utilities Specific

These rules are ALWAYS ACTIVE for all specialized agent adapter and integration modules within the agent subsystem.

### Rules

- **R-AGNT-001** MAY: Agent implementations MAY integrate auxiliary utilities for specific protocol formatting or file operations when external runtime requirements necessitate specialized behavior.
- **R-AGNT-002** MANDATORY: When authoring a new agent module, extend AgentsMdAgent and implement specialized hooks rather than reimplementing execution workflows.
- **R-AGNT-003** MANDATORY: Verify that internal module imports maintain subsystem boundaries and adhere to shared agent interface expectations.
- **R-AGNT-004** MANDATORY (DISCOVERY POLICY): Omit all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.
- **R-AGNT-005** MANDATORY (LOCK-VERSION GROUNDING): Before writing code that uses a versioned library, execute in order: 1. Find the dependency manifest in the repo (ranges, not installed). 2. Identify build tool. 3. Inspect repository lock or resolution artifact for exact resolved version. 4. Look up official documentation/changelog/public API reference for that exact version. 5. Confirm every API/class/function exists in that version's docs before use. 6. Re-run steps 3-5 per dependency at point of use for version-sensitive behavior.

### Verify

```bash
# Discover and run the project static analysis suite to verify that agent modules import AgentsMdAgent.
# Discover and run the project test suite to validate that all agent implementations satisfy regression and integration tests.
```

**Accept when:**
- All agent integration modules resolve and import AgentsMdAgent without static analysis errors.
- The project test runner executes and passes all test suites covering the agent subsystem.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests lacking base module inheritance in agent components are blocked and require refactoring to extend AgentsMdAgent unless an approved architecture review exception is obtained.
</enforcement>
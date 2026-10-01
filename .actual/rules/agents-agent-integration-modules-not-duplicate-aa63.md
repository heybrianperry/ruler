# Adoption of AgentsMdAgent as Base Implementation for Agent Modules: Agent Integration Modules Not Duplicate Core

These rules are ALWAYS ACTIVE for all specialized agent adapter and integration modules within the agent subsystem.

### Rules

- **R-AG-001** MUST_NOT: Agent integration modules MUST NOT duplicate core orchestration routines provided by AgentsMdAgent.

### Verify

```bash
# Discover and run the project static analysis suite to verify that agent modules import AgentsMdAgent.
# Discover and run the project test suite to validate that all agent implementations satisfy regression and integration tests.
```

**Accept when:**
- All agent integration modules resolve and import AgentsMdAgent without static analysis errors.
- The project test runner executes and passes all test suites covering the agent subsystem.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated static analysis and code reviews enforce these requirements; violations require refactoring to extend AgentsMdAgent unless an explicit architecture review exception is approved.
</enforcement>
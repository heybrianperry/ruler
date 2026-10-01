# Adoption of AgentsMdAgent as Base Implementation for Agent Modules: Agent Integration Modules Adopt Agentsmdagent Their

These rules are ALWAYS ACTIVE for all specialized agent adapter and integration modules within the agent subsystem.

### Rules

- **R-AGT-001** MUST: Agent integration modules MUST adopt AgentsMdAgent as their shared foundation to ensure uniform lifecycle execution.

### Verify

```bash
# Discover and run the project static analysis and test suites covering the agent subsystem
# (Commands to be derived from the project repository configuration)
```

**Accept when:**
- All agent integration modules resolve and import AgentsMdAgent without static analysis errors.
- The project test runner executes and passes all test suites covering the agent subsystem.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
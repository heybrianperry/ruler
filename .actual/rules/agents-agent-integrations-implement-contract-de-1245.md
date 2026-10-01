# Adoption of IAgent Interface Module for Agent Implementations: Agent Integrations Implement Contract Defined Internal

These rules are ALWAYS ACTIVE for all agent integration modules, agent utility routines, and agent orchestration dispatchers within the agent subsystem, including all new coding assistant or automation agent integrations being introduced to the codebase.

### Rules

- **R-AGENT-001** MUST: All agent integrations MUST implement the contract defined by the internal IAgent interface module as their public architectural boundary.
- **R-AGENT-002** MUST: Ensure that all newly introduced agent providers provide concrete implementations satisfying the IAgent contract before exposing them in the agent index export.
- **R-AGENT-003** SHOULD: Centralize shared capabilities and foundational file system interactions in an abstract base agent class rather than duplicated across individual agent implementations.

### Verify

```bash
# Discover the repository typecheck or build script from the project manifest and execute it
# Example: npm run build / tsc --noEmit
# Discover the repository test runner script and execute the agent test suite
# Example: npm test
```

**Accept when:**
- All agent integration classes strictly implement the IAgent contract with zero type checking or compilation errors.
- Agent orchestration workflows invoke diverse agent implementations solely through the IAgent contract.

<enforcement>
Verification via automated CI build checks and peer code review is mandatory. Code containing agent implementations that do not satisfy the IAgent contract or direct consumer imports of concrete agent implementation classes in orchestration modules will be rejected.
</enforcement>
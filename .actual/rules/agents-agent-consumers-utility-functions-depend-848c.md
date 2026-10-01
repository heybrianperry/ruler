# Adoption of IAgent Interface Module for Agent Implementations: Agent Consumers Utility Functions Depend Exclusively

These rules are ALWAYS ACTIVE for agent integration modules, agent utility routines, and agent orchestration dispatchers within the agent subsystem, as well as new coding assistant or automation agent integrations being introduced to the codebase.

### Rules

- **R-AGT-001** MUST: Agent consumers and utility functions MUST depend exclusively upon the IAgent contract rather than concrete agent implementation classes when referencing, orchestrating, or dispatching agents.
- **R-AGT-002** MUST: Ensure that all newly introduced agent providers provide concrete implementations satisfying the IAgent contract before exposing them in the agent index export.
- **R-AGT-003** SHOULD: Centralize shared capabilities and foundational file system interactions in an abstract base agent class rather than duplicated across individual agent implementations.

### Verify

```bash
# Discover the repository typecheck or build script from the project manifest and execute it
# Example: pnpm build / npm run build / cargo check
# Discover the repository test runner script and execute the agent test suite
# Example: pnpm test / npm test
```

**Accept when:**
- All agent integration classes strictly implement the IAgent contract with zero type checking or compilation errors.
- Agent orchestration workflows invoke diverse agent implementations solely through the IAgent contract.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
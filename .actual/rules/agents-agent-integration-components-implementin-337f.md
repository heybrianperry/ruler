# AgentsMdAgent Module Adoption for Agent Implementations: Agent Integration Components Implementing Autonomous Interactive

These rules are ALWAYS ACTIVE for agent integration components implementing autonomous or interactive agent capabilities, and shared agent configuration, document generation, and interface definitions within the agent subsystem.

### Rules

- **R-AGNT-001** MUST: Agent integration components implementing autonomous or interactive agent capabilities MUST conform to the IAgent contract when exposing public agent operational interfaces.
- **R-AGNT-002** MANDATORY: Discover the project test runner configuration from the repository manifest and execute the test suite covering agent integration modules.
- **R-AGNT-003** MANDATORY: Discover the project static analysis and linting configuration from the repository manifest and execute validation across all agent integration components.

### Verify

```bash
# Discover the project test runner configuration from the repository manifest and execute the test suite covering agent integration modules.
# Discover the project static analysis and linting configuration from the repository manifest and execute validation across all agent integration components.
```

**Accept when:**
- All agent integration modules successfully instantiate and inherit from AgentsMdAgent without contract violations.
- All agent test suites pass cleanly with zero linting and type-checking diagnostics reported across agent implementations.

<enforcement>
Claude Code MUST NOT skip or defer verification. All agent integration components must conform to the IAgent contract and extend AgentsMdAgent.
</enforcement>
# IAgent Internal Interface Contract for Agent Integrations: Concrete Agent Modules Implement Internal Serialization

These rules are ALWAYS ACTIVE for all agent integration modules, agent adapters, and agent utility implementations within the agent subsystem.

### Rules

- **R-AGT-001** MAY: Concrete agent modules MAY implement internal serialization, configuration parsing, or environment validation routines privately, provided all external interactions remain governed by the IAgent contract.
- **R-AGT-002** MANDATORY: When creating a new agent integration, implement the IAgent contract and register the implementation within the primary agent registry index module.
- **R-AGT-003** MANDATORY: Agent integrations requiring specialized configuration parsing must encapsulate parsing logic within private implementation methods while exposing standard IAgent lifecycle behaviors.

### Verify

```bash
# Discover and execute the repository static type checker and linter across the agent subsystem
# (Command must be derived from the repository project configuration)

# Discover and run the project test suite targeting agent integration test suites
# (Command must be derived from the repository project configuration)
```

**Accept when:**
- Static type analysis confirms that all concrete agent integration modules implement the IAgent interface contract without type errors.
- The project test suite passes with zero failures across all agent integration and utility test suites.

<enforcement>
Claude Code MUST NOT skip or defer verification. All new or modified agent integration modules must be verified via automated static type analysis, linting, and peer code review.
</enforcement>
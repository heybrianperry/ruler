# IAgent Internal Interface Contract for Agent Integrations: Shared Agent Execution Logic Boilerplate Lifecycle

These rules are ALWAYS ACTIVE for all agent integration modules, agent adapters, and agent utility implementations within the agent subsystem.

### Rules

- **R-AGT-001** SHOULD: Shared agent execution logic and boilerplate lifecycle handling SHOULD be factored into abstract base modules conforming to IAgent to prevent redundant code across concrete implementations.

### Verify

```bash
# Discover and execute the repository static type checker and linter across the agent subsystem to verify interface conformance with the IAgent contract.
# Discover and run the project test suite targeting agent integration test suites to validate contract adherence.
```

**Accept when:**
- Static type analysis confirms that all concrete agent integration modules implement the IAgent interface contract without type errors.
- The project test suite passes with zero failures across all agent integration and utility test suites.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory via automated static type analysis, linting, and peer code review.
</enforcement>
# IAgent Internal Interface Contract for Agent Integrations: Agent Integration Modules Subsystem Implement Consume

These rules are ALWAYS ACTIVE for all agent integration modules, agent adapters, and agent utility implementations within the agent subsystem.

### Rules

- **R-AGT-001** MUST: All agent integration modules in the agent subsystem MUST implement or consume the IAgent contract as their primary interface boundary.

### Verify

```bash
# Discover and execute the repository static type checker and linter across the agent subsystem to verify interface conformance with the IAgent contract.
# Discover and run the project test suite targeting agent integration test suites to validate contract adherence.
```

**Accept when:**
- Static type analysis confirms that all concrete agent integration modules implement the IAgent interface contract without type errors.
- The project test suite passes with zero failures across all agent integration and utility test suites.

<enforcement>
Verification via automated static type analysis, linting, and test suites is mandatory. Pull requests introducing non-conforming implementations are blocked from merging, and static analysis failures trigger immediate build termination.
</enforcement>
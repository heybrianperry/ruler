# IAgent Internal Interface Contract for Agent Integrations: When Consuming External Libraries Supporting Agent

These rules are ALWAYS ACTIVE for all agent integration modules, agent adapters, and agent utility implementations within the agent subsystem.

### Rules

- **R-AGNT-001** MUST: When consuming external libraries supporting agent integrations, the consumer MUST inspect the dependency manifest and authoritative lock artifact to resolve the exact locked version before implementation.
- **R-AGNT-002** MUST: All agent implementations and utility modules MUST adopt and implement the internal IAgent contract.
- **R-AGNT-003** MUST: When creating a new agent integration, the consumer MUST implement the IAgent contract and register the implementation within the primary agent registry index module.
- **R-AGNT-004** MUST: Agent integrations requiring specialized configuration parsing MUST encapsulate parsing logic within private implementation methods while exposing standard IAgent lifecycle behaviors.

### Verify

```bash
# Discover and execute the repository static type checker and linter across the agent subsystem to verify interface conformance
# Discover and run the project test suite targeting agent integration test suites to validate contract adherence
```

**Accept when:**
- Static type analysis confirms that all concrete agent integration modules implement the IAgent interface contract without type errors.
- The project test suite passes with zero failures across all agent integration and utility test suites.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated static type analysis and linting within CI pipelines verify interface conformance, and pull requests introducing non-conforming implementations are blocked from merging.
</enforcement>
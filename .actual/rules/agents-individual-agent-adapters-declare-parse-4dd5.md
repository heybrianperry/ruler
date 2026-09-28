# AgentsMdAgent Internal Module Adoption for Agent Adapters: Individual Agent Adapters Declare Parse Specific

These rules are ALWAYS ACTIVE for all agent adapter implementations and modules providing interfaces or base classes for AI agent interaction and markdown configuration.

### Rules

- **R-AGT-001** MAY: Individual agent adapters MAY declare and parse agent-specific auxiliary configuration structures when their target runtime requires unique parameters not handled by the shared AgentsMdAgent base module.
- **R-AGT-002** MUST: When creating a new agent adapter, inherit from the internal AgentsMdAgent class and implement all abstract lifecycle and configuration methods mandated by the IAgent contract.
- **R-AGT-003** MUST: Delegate generic configuration file parsing and markdown synchronization to base class methods, restricting adapter-specific code to tool-specific options and execution semantics.

### Verify

```bash
# Discover and execute the static analysis, type verification, and architectural module boundary test suites
# (Repository-specific script discovery required; execute type checking and linting across all agent modules)
```

**Accept when:**
- All agent adapter modules cleanly resolve dependencies and pass type checking while extending the internal AgentsMdAgent module.
- Module boundary checks confirm that no agent adapter module bypasses the IAgent interface or provides duplicate core agent lifecycle implementations.

<enforcement>
Verification via static type checking and module boundary lint rules is mandatory for all pull requests modifying or introducing agent adapters.
</enforcement>
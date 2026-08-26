# Adopt AgentsMdAgent Base Class for Agent Implementation Extensibility: Agent Implementations Not Bypass Agentsmdagent Base

These rules are ALWAYS ACTIVE for all agent implementations in the `src/agents/` directory and any new agent types added to the codebase.

### Rules

- **R-AGENT-001** MUST NOT: Agent implementations MUST NOT bypass the AgentsMdAgent base class by implementing agents from scratch, as this breaks architectural consistency and duplicates shared functionality.
- **R-AGENT-002** MUST: All new agent implementations added to the agents package MUST extend AgentsMdAgent and implement the IAgent interface.
- **R-AGENT-003** MUST: Agent implementations MUST use relative imports to maintain module cohesion within the agents package.
- **R-AGENT-004** SHOULD: When an agent requires functionality not present in the base class, first evaluate whether the functionality is general enough to belong in AgentsMdAgent before implementing it in the derived class.
- **R-AGENT-005** SHOULD: Developers SHOULD examine existing agent implementations to understand available methods in AgentsMdAgent and override only methods requiring agent-specific behavior.
- **R-AGENT-006** MAY: Exception EX-001 MAY be granted for creating a fundamentally different agent abstraction that cannot reasonably extend AgentsMdAgent due to incompatible architectural assumptions, subject to architecture review and documentation.

### Verify

```bash
# Discover and run the project's type checker to verify all agent implementations
# correctly extend AgentsMdAgent and implement IAgent
echo "Running type checker on agents package..."
# (Exact command depends on project's build tool and tsconfig)

# Discover and run the project's test runner for the agents package
echo "Running test suite for agents package..."
# (Exact command depends on project's test framework)

# Discover and run the project's linter to verify import patterns
# and inheritance structure conform to this ADR
echo "Running linter on agents package..."
# (Exact command depends on project's linting configuration)
```

**Accept when:**
- All agent implementations in the agents package extend AgentsMdAgent and the type checker reports no inheritance or interface violations
- All tests for agent implementations pass, demonstrating that the inheritance pattern does not break agent functionality
- Code review confirms that new agent implementations follow the established pattern and do not bypass the base class
- Linting rules detect no agents that fail to import or extend the base class

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checker failures, test failures, or linting violations block acceptance. Exceptions require documented architecture review approval.
</enforcement>
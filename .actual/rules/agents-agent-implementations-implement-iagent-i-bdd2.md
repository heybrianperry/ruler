# Adopt AgentsMdAgent Base Class for Agent Implementation Extensibility: Agent Implementations Implement Iagent Interface Ensure

These rules are ALWAYS ACTIVE for all agent implementations in the `src/agents/` directory and any new agent types added to the codebase.

### Rules

- **R-AGENT-001** SHOULD: Agent implementations SHOULD implement the IAgent interface to ensure they fulfill the agent contract expected by the system.
- **R-AGENT-002** MUST: All new agent implementations added to the agents package MUST extend AgentsMdAgent as the base class.
- **R-AGENT-003** SHOULD: Agent implementations SHOULD use relative imports to maintain module cohesion within the agents package.
- **R-AGENT-004** SHOULD: When an agent requires functionality not present in the base class, developers SHOULD first evaluate whether the functionality is general enough to belong in AgentsMdAgent before implementing it in the derived class.
- **R-AGENT-005** MUST: TypeScript strict type checking MUST be enabled to ensure agent implementations correctly extend AgentsMdAgent and implement IAgent at compile time.
- **R-AGENT-006** MAY: Developers MAY create a fundamentally different agent abstraction that cannot reasonably extend AgentsMdAgent only when documented as exception EX-001 with architectural justification.

### Verify

```bash
# Discover and run the project's type checker to verify all agent implementations
# correctly extend AgentsMdAgent and implement IAgent
find . -name "tsconfig.json" -o -name "package.json" | head -1 | xargs grep -l "typescript" && \
  npx tsc --noEmit

# Discover and run the project's test runner for the agents package
find . -name "package.json" | xargs grep -l "jest\|vitest\|mocha" | head -1 && \
  npm test -- src/agents

# Discover and run the project's linter to verify import patterns and inheritance
find . -name ".eslintrc*" -o -name "eslint.config.*" | head -1 && \
  npx eslint src/agents --max-warnings 0
```

**Accept when:**
- All agent implementations in the agents package extend AgentsMdAgent and the type checker reports no inheritance or interface violations
- All tests for agent implementations pass, demonstrating that the inheritance pattern does not break agent functionality
- Code review confirms that new agent implementations follow the established pattern and do not bypass the base class
- Linting passes with no violations related to import patterns or inheritance structure

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checker failures, test failures, or linting violations block acceptance. Exception process requires documented architectural justification and architecture review approval before proceeding.
</enforcement>
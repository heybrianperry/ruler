# Adopt AgentsMdAgent Base Class for Agent Implementation Extensibility: Agent Implementations Import Additional Libraries Node

These rules are ALWAYS ACTIVE for all agent implementations in the `src/agents/` directory and any new agent types added to the codebase.

### Rules

- **R-AGENT-001** MAY: Agent implementations MAY import additional libraries (Node.js core modules, third-party libraries, or internal utilities) as needed for agent-specific functionality.
- **R-AGENT-002** MUST: All agent implementations MUST extend AgentsMdAgent as their base class.
- **R-AGENT-003** MUST: All agent implementations MUST implement the IAgent interface contract.
- **R-AGENT-004** SHOULD: Agent implementations SHOULD use relative imports to maintain module cohesion within the agents package.
- **R-AGENT-005** SHOULD: When an agent requires functionality not present in the base class, first evaluate whether the functionality is general enough to belong in AgentsMdAgent. If it benefits multiple agents, add it to the base class. If it is agent-specific, implement it in the derived class.
- **R-AGENT-006** SHOULD: Override only the methods in AgentsMdAgent that require agent-specific behavior.
- **R-AGENT-007** SHOULD: Use TypeScript's type system with strict type checking enabled to ensure agent implementations correctly implement the IAgent interface.
- **R-AGENT-008** MAY: An agent implementation MAY deviate from extending AgentsMdAgent only under exception EX-001 when creating a fundamentally different agent abstraction that cannot reasonably extend AgentsMdAgent due to incompatible architectural assumptions. Such exceptions MUST be documented with architectural rationale.

### Verify

```bash
# Discover and run the project's type checker to verify all agent implementations
# correctly extend AgentsMdAgent and implement IAgent
find . -name 'tsconfig.json' -o -name 'package.json' | head -1 | xargs grep -l 'typescript' | xargs -I {} sh -c 'npm run type-check || npx tsc --noEmit'

# Discover and run the project's test runner for the agents package
find . -name 'package.json' -exec grep -l 'jest\|vitest\|mocha' {} \; | head -1 | xargs -I {} sh -c 'npm test -- src/agents'

# Discover and run the project's linter to verify import patterns and inheritance
find . -name '.eslintrc*' -o -name 'eslint.config.*' | head -1 | xargs -I {} sh -c 'npm run lint -- src/agents'
```

**Accept when:**
- All agent implementations in the agents package extend AgentsMdAgent and the type checker reports no inheritance or interface violations
- All tests for agent implementations pass, demonstrating that the inheritance pattern does not break agent functionality
- Code review confirms that new agent implementations follow the established pattern and do not bypass the base class
- Linting passes with no violations of import patterns or inheritance structure

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checker failures, test failures, or linting violations block acceptance. Exception EX-001 requires explicit documentation and architecture review approval before code integration.
</enforcement>
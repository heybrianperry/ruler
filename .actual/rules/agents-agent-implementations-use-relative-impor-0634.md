# Adopt AgentsMdAgent Base Class for Agent Implementation Extensibility: Agent Implementations Use Relative Imports Reference

These rules are ALWAYS ACTIVE for all agent implementations in the agents package and any code that extends or modifies the agent system architecture.

### Rules

- **R-AGENTS-001** MUST: Agent implementations MUST use relative imports to reference AgentsMdAgent and IAgent to maintain module cohesion within the agents package.
- **R-AGENTS-002** MUST: All new agent implementations added to the agents package MUST extend AgentsMdAgent and implement the IAgent interface.
- **R-AGENTS-003** MUST: Agent implementations MUST override only the methods that require agent-specific behavior; shared functionality MUST be implemented in AgentsMdAgent.
- **R-AGENTS-004** SHOULD: When an agent requires functionality not present in the base class, first evaluate whether the functionality is general enough to belong in AgentsMdAgent before implementing it in the derived class.
- **R-AGENTS-005** SHOULD: Use TypeScript's type system with strict type checking enabled to ensure agent implementations correctly implement the IAgent interface.

### Verify

```bash
# Discover and run the project's type checker to verify all agent implementations
# correctly extend AgentsMdAgent and implement IAgent
find . -name 'tsconfig.json' -o -name 'tsconfig*.json' | head -1 | xargs -I {} dirname {} | xargs -I {} sh -c 'cd {} && npm run type-check || yarn type-check || tsc --noEmit'

# Discover and run the project's test runner for the agents package
find . -name 'package.json' | xargs grep -l '"test"' | head -1 | xargs -I {} dirname {} | xargs -I {} sh -c 'cd {} && npm test -- agents || yarn test agents'

# Discover and run the project's linter to verify import patterns and inheritance structure
find . -name '.eslintrc*' -o -name 'eslint.config.*' | head -1 | xargs -I {} dirname {} | xargs -I {} sh -c 'cd {} && npm run lint || yarn lint'
```

**Accept when:**
- All agent implementations in the agents package extend AgentsMdAgent and the type checker reports no inheritance or interface violations
- All tests for agent implementations pass, demonstrating that the inheritance pattern does not break agent functionality
- Code review confirms that new agent implementations follow the established pattern and do not bypass the base class
- Linting rules confirm import patterns and inheritance structure conform to this ADR

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checker failures, test failures, or linting violations block acceptance. Exception process requires explicit documentation and architecture review approval before proceeding.
</enforcement>
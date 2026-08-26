# Adopt AgentsMdAgent Base Class for Agent Implementation Extensibility: Agent Implementations Extend Agentsmdagent Base Class

These rules are ALWAYS ACTIVE for all agent implementations in the codebase, including new agent types added to the agents package and any refactoring or modification of existing agent implementations.

### Rules

- **R-AGENT-001** MUST: All agent implementations MUST extend the AgentsMdAgent base class to inherit shared agent functionality and maintain architectural consistency.
- **R-AGENT-002** MUST: All agent implementations MUST implement the IAgent interface contract to enable polymorphic usage throughout the system.
- **R-AGENT-003** SHOULD: When implementing a new agent, examine existing agent implementations to understand the methods available in AgentsMdAgent and override only methods that require agent-specific behavior.
- **R-AGENT-004** SHOULD: If an agent requires functionality not present in the base class, first evaluate whether the functionality is general enough to belong in AgentsMdAgent before implementing it in the derived class.
- **R-AGENT-005** SHOULD: Use TypeScript's type system with strict type checking enabled to ensure agent implementations correctly implement the IAgent interface and catch violations at compile time.
- **R-AGENT-006** SHOULD: Use relative imports within the agents package to maintain strong module cohesion and keep the agent system self-contained.
- **R-AGENT-007** MAY: Create a fundamentally different agent abstraction that cannot reasonably extend AgentsMdAgent only when documented as exception EX-001 with architectural justification.

### Verify

```bash
# Discover and run the project's type checker to verify all agent implementations
# correctly extend AgentsMdAgent and implement IAgent
find . -name 'tsconfig.json' -o -name 'package.json' | head -1 | xargs grep -l 'typescript' | xargs -I {} sh -c 'npm run type-check || npx tsc --noEmit'

# Discover and run the project's test runner for the agents package
find . -name 'package.json' | xargs grep -l 'jest\|vitest\|mocha' | head -1 | xargs -I {} sh -c 'npm test -- src/agents'

# Discover and run the project's linter to verify import patterns and inheritance
find . -name '.eslintrc*' -o -name 'eslint.config.*' | head -1 | xargs -I {} sh -c 'npx eslint src/agents --max-warnings 0'

# Verify all agent files in src/agents extend AgentsMdAgent
grep -r 'extends AgentsMdAgent' src/agents/ | wc -l
```

**Accept when:**
- All agent implementations in the agents package extend AgentsMdAgent and the type checker reports no inheritance or interface violations
- All tests for agent implementations pass, demonstrating that the inheritance pattern does not break agent functionality
- Code review confirms that new agent implementations follow the established pattern and do not bypass the base class
- The linter reports no violations in import patterns and inheritance structure

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checker failures, test failures, or linting violations block acceptance. Exceptions require documented architectural justification and architecture review approval.
</enforcement>
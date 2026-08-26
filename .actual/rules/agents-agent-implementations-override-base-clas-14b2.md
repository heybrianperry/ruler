# Adopt AgentsMdAgent Base Class for Agent Implementation Extensibility: Agent Implementations Override Base Class Methods

These rules are ALWAYS ACTIVE for all agent implementations in the `src/agents/` directory and any new agent types added to the codebase.

### Rules

- **R-AGENTS-001** SHOULD: Agent implementations SHOULD override base class methods to provide agent-specific behavior while delegating common functionality to the parent class.

### Verify

```bash
# Discover the project's static analysis configuration and execute the type checker
# to verify all agent implementations correctly extend AgentsMdAgent and implement IAgent
find . -name "tsconfig.json" -o -name "tsc.config.*" | head -1 | xargs -I {} dirname {} | xargs -I {} sh -c 'cd {} && npx tsc --noEmit'

# Discover the project's test runner and execute the test suite for the agents package
find . -name "package.json" | xargs grep -l '"test"' | head -1 | xargs -I {} dirname {} | xargs -I {} sh -c 'cd {} && npm test -- src/agents'

# Discover the project's linting configuration and execute the linter
find . -name ".eslintrc*" -o -name "eslint.config.*" | head -1 | xargs -I {} dirname {} | xargs -I {} sh -c 'cd {} && npx eslint src/agents --max-warnings 0'
```

**Accept when:**
- All agent implementations in the agents package extend AgentsMdAgent and the type checker reports no inheritance or interface violations
- All tests for agent implementations pass, demonstrating that the inheritance pattern does not break agent functionality
- Code review confirms that new agent implementations follow the established pattern and do not bypass the base class
- Linting rules detect no agents that fail to import or extend the base class

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checker failures, test failures, or linting violations block acceptance. Exception process requires architecture review and documentation in code comments and ADR exception log.
</enforcement>
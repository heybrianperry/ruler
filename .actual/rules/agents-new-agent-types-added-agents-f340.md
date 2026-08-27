# Adopt AgentsMdAgent Base Class for Agent Implementations: New Agent Types Added Agents Subsystem

These rules are ALWAYS ACTIVE for all TypeScript classes in the `src/agents/` directory that implement agent behavior, including new agent types being added to the agents subsystem and refactoring of existing agent implementations.

### Rules

- **R-AGENTS-001** MUST: New agent types added to the agents subsystem MUST conform to the interface contract established by AgentsMdAgent.

### Verify

```bash
# Discover the project's static analysis or linting configuration and execute the verification script to check that all agent implementation files import AgentsMdAgent

# Discover the project's type checking tooling and verify that all classes in the agents directory conform to the base class interface contract

# Discover the project's test suite and run agent subsystem tests to verify that all agent implementations satisfy the shared interface requirements
```

**Accept when:**
- All agent implementation files in the agents directory successfully import and extend or compose AgentsMdAgent base class
- Type checking passes with no interface contract violations in agent implementations
- Agent subsystem tests pass, confirming all implementations satisfy shared interface requirements

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis tools verify AgentsMdAgent import presence in all agent implementation files. Type checking enforces interface contract compliance at build time. Code review process checks that new agent types extend the base class. Automated tests validate that agent implementations conform to expected interface. Build fails if type checking detects interface contract violations. Code review blocks merge of agent implementations that do not extend AgentsMdAgent. Static analysis warnings flag missing base class imports in agent files.
</enforcement>
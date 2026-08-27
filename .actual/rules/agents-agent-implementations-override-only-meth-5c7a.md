# Adopt AgentsMdAgent Base Class for Agent Implementations: Agent Implementations Override Only Methods Necessary

These rules are ALWAYS ACTIVE for all TypeScript classes in the `src/agents/` directory that implement agent behavior, including new agent types being added to the agents subsystem and refactoring of existing agent implementations.

### Rules

- **R-AGENTS-001** SHOULD: Agent implementations SHOULD override only the methods necessary for specialized behavior, preserving base class defaults where applicable.

### Verify

```bash
# Verify that all agent implementation files import AgentsMdAgent
grep -r "AgentsMdAgent" src/agents/ --include="*.ts" | grep -E "(import|extends)" | wc -l

# Run type checking to verify interface contract compliance
npm run type-check

# Run agent subsystem tests to verify implementations satisfy shared interface
npm run test -- src/agents/
```

**Accept when:**
- All agent implementation files in the `src/agents/` directory successfully import and extend AgentsMdAgent base class
- Type checking passes with no interface contract violations in agent implementations
- Agent subsystem tests pass, confirming all implementations satisfy shared interface requirements

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis tools verify AgentsMdAgent import presence in all agent implementation files. Type checking enforces interface contract compliance at build time. Code review process checks that new agent types extend the base class. Build fails if type checking detects interface contract violations. Code review blocks merge of agent implementations that do not extend AgentsMdAgent.
</enforcement>
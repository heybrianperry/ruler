# Agent Implementations Extend AbstractAgent Base Class: Agent Implementations Override Only Methods Necessary

These rules are ALWAYS ACTIVE for all TypeScript classes in the agents module that implement agent behavior for AI service backends, including new agent implementations added to support additional AI services or orchestration patterns.

### Rules

- **R-AGENT-001** SHOULD: Agent implementations SHOULD override only the methods necessary for their specific backend integration, relying on AbstractAgent for common functionality.

### Verify

```bash
# Discover the agents module location in the repository and inspect all class files to confirm they import and extend the base class
find src/agents -name '*.ts' -type f ! -path '*/test/*' ! -path '*/mock/*' | xargs grep -l 'extends AbstractAgent'

# Locate the project's static analysis or linting configuration and execute the type checker to verify inheritance contracts are satisfied
npx tsc --noEmit

# Identify the test suite for the agents module and run tests to confirm polymorphic behavior works correctly across all agent implementations
npm test -- src/agents
```

**Accept when:**
- All agent implementation files in the agents module contain import statements referencing the base class and use `extends` syntax
- Type checking passes without errors related to missing abstract method implementations or contract violations
- Agent module tests demonstrate that all concrete agent types can be instantiated and used polymorphically through the common interface

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compilation failures or test failures indicate rule violations that must be resolved before proceeding.
</enforcement>
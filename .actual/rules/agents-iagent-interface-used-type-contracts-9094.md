# Agent Implementations Extend AbstractAgent Base Class: Iagent Interface Used Type Contracts Where

These rules are ALWAYS ACTIVE for all TypeScript classes in the agents module that implement agent behavior for AI service backends, and for new agent implementations added to support additional AI services or orchestration patterns.

### Rules

- **R-AGENT-001** MUST: All agent implementation classes in the agents module extend AbstractAgent base class.
- **R-AGENT-002** SHOULD: The IAgent interface SHOULD be used for type contracts where polymorphic agent handling is required outside the agents module.
- **R-AGENT-003** MUST: New agent implementations override only the abstract methods and backend-specific logic required by their service integration; concrete methods from AbstractAgent should be inherited unless there is documented justification for override.
- **R-AGENT-004** MUST: All new agent types be added to the agents module index to ensure they are discoverable by orchestration code.
- **R-AGENT-005** MUST: Agent implementations that cannot accommodate AbstractAgent inheritance due to fundamentally different lifecycle or contract requirements must document the exception with clear rationale and submit an architecture review.

### Verify

```bash
# Discover the agents module location and inspect all class files
find src/agents -name '*.ts' -type f | xargs grep -l 'class.*Agent' | xargs grep -E '(extends AbstractAgent|implements IAgent)'

# Locate and run the TypeScript type checker
npx tsc --noEmit

# Run the agents module test suite
npm test -- src/agents
```

**Accept when:**
- All agent implementation files in the agents module contain import statements referencing AbstractAgent and use extends syntax
- TypeScript compilation passes without errors related to missing abstract method implementations or contract violations
- Agent module tests demonstrate that all concrete agent types can be instantiated and used polymorphically through the common interface
- All new agent types are registered in the agents module index

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler type checking is mandatory and will fail if agent classes do not properly extend AbstractAgent or implement required methods. Code review must verify new agent implementations follow the established pattern.
</enforcement>
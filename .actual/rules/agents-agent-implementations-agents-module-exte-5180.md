# Agent Implementations Extend AbstractAgent Base Class: Agent Implementations Agents Module Extend Abstractagent

These rules are ALWAYS ACTIVE for all agent implementation files in the agents module that implement agent behavior for AI service backends.

### Rules

- **R-AGENT-001** MUST: All agent implementations in the agents module MUST extend the AbstractAgent base class to inherit shared behavior and maintain polymorphic compatibility.
- **R-AGENT-002** MUST: When creating a new agent implementation, examine AbstractAgent to understand which methods are abstract (must be implemented) versus concrete (can be inherited), and override only what is necessary for the specific backend.
- **R-AGENT-003** MUST: Add new agent implementations to the agents module index to ensure they are discoverable by orchestration code.
- **R-AGENT-004** SHOULD: Consider whether the agent implementation requires state management, lifecycle hooks, or error handling patterns that AbstractAgent may already provide before implementing custom solutions.
- **R-AGENT-005** MAY: A specialized agent may require fundamentally different lifecycle or contract that cannot be accommodated by AbstractAgent (Exception EX-001), provided the exception is documented in the agent implementation file with clear rationale and approved through architecture review.

### Verify

```bash
# Discover the agents module location and inspect all class files
find src/agents -name '*.ts' -type f | grep -v test | grep -v spec

# Verify all agent implementations import and extend AbstractAgent
grep -l "extends AbstractAgent" src/agents/*.ts

# Run TypeScript compiler to verify inheritance contracts
npx tsc --noEmit

# Run the agents module test suite
npm test -- src/agents
```

**Accept when:**
- All agent implementation files in the agents module contain import statements referencing AbstractAgent and use extends syntax
- TypeScript compiler passes without errors related to missing abstract method implementations or contract violations
- Agent module tests demonstrate that all concrete agent types can be instantiated and used polymorphically through the common interface
- New agent implementations are registered in the agents module index

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compilation failure or missing AbstractAgent inheritance is a hard blocker for agent implementation code.
</enforcement>
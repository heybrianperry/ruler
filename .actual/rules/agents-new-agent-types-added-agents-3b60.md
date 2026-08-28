# Agent Implementations Extend AbstractAgent Base Class: New Agent Types Added Agents Module

These rules are ALWAYS ACTIVE for all TypeScript classes in the agents module that implement agent behavior for AI service backends, and for new agent implementations added to support additional AI services or orchestration patterns.

### Rules

- **R-AGENT-001** MUST: New agent types MUST be added to the agents module index for centralized export and discovery.
- **R-AGENT-002** MUST: All new agent implementations MUST extend AbstractAgent base class.
- **R-AGENT-003** MUST: New agent implementations MUST implement all abstract methods defined by AbstractAgent.
- **R-AGENT-004** SHOULD: When creating a new agent implementation, examine AbstractAgent to understand which methods are abstract (must be implemented) versus concrete (can be inherited), and override only what is necessary for the specific backend.
- **R-AGENT-005** SHOULD: Consider whether the agent implementation requires state management, lifecycle hooks, or error handling patterns that AbstractAgent may already provide before implementing custom solutions.
- **R-AGENT-006** MAY: A specialized agent may require fundamentally different lifecycle or contract that cannot be accommodated by AbstractAgent (Exception EX-001), provided the exception is documented in the agent implementation file with clear rationale and approved through architecture review.

### Verify

```bash
# Discover the agents module location in the repository and inspect all class files
find src/agents -name '*.ts' -type f | xargs grep -l 'extends AbstractAgent'

# Locate the project's static analysis or linting configuration and execute the type checker
npx tsc --noEmit

# Identify the test suite for the agents module and run tests
npm test -- src/agents
```

**Accept when:**
- All agent implementation files in the agents module contain import statements referencing the base class and use extends syntax
- Type checking passes without errors related to missing abstract method implementations or contract violations
- Agent module tests demonstrate that all concrete agent types can be instantiated and used polymorphically through the common interface
- New agent types are exported from the agents module index

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compilation failure or missing index exports constitute rule violations. Code review MUST reject pull requests that add agent implementations without following the established inheritance pattern.
</enforcement>
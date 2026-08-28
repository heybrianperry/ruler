# Agent Implementations Extend AbstractAgent Base Class: Agent Implementations Not Bypass Abstractagent Implementing

These rules are ALWAYS ACTIVE for all TypeScript classes in the agents module that implement agent behavior for AI service backends, including new agent implementations added to support additional AI services or orchestration patterns.

### Rules

- **R-AGENT-001** MUST NOT: Agent implementations MUST NOT bypass AbstractAgent by implementing IAgent directly unless explicitly documented as an exception.
- **R-AGENT-002** MUST: When creating a new agent implementation, examine AbstractAgent to understand which methods are abstract (must be implemented) versus concrete (can be inherited).
- **R-AGENT-003** MUST: Override only what is necessary for your specific backend in new agent implementations.
- **R-AGENT-004** MUST: Add new agents to the agents module index to ensure they are discoverable by orchestration code.
- **R-AGENT-005** SHOULD: Consider whether your agent implementation requires state management, lifecycle hooks, or error handling patterns that AbstractAgent may already provide before implementing custom solutions.
- **R-AGENT-006** SHOULD: Document any approved exceptions in the agent implementation file with clear rationale.

### Verify

```bash
# Discover the agents module location and inspect all class files
find src/agents -name '*.ts' -type f | xargs grep -l 'class.*Agent' | while read file; do
  echo "Checking $file for AbstractAgent inheritance..."
  grep -E '(extends AbstractAgent|implements IAgent)' "$file" || echo "WARNING: $file may not follow inheritance pattern"
done

# Run TypeScript compiler to verify inheritance contracts
npx tsc --noEmit

# Run agent module tests to confirm polymorphic behavior
npm test -- src/agents
```

**Accept when:**
- All agent implementation files in the agents module contain import statements referencing AbstractAgent and use extends syntax
- TypeScript compilation passes without errors related to missing abstract method implementations or contract violations
- Agent module tests demonstrate that all concrete agent types can be instantiated and used polymorphically through the common interface
- Any direct IAgent implementations are documented as approved exceptions with clear rationale

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler type checking and code review are mandatory enforcement points. Violations will cause compilation failure or require documented exception approval.
</enforcement>
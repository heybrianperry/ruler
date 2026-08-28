# Agent Implementations Extend AbstractAgent Base Class: Agent Implementations Use Relative Path Imports

These rules are ALWAYS ACTIVE for all TypeScript classes in the agents module that implement agent behavior for AI service backends.

### Rules

- **R-AGENT-001** MUST: Agent implementations MUST use relative path imports to reference AbstractAgent from within the agents module.

### Verify

```bash
# Discover the agents module location in the repository and inspect all class files to confirm they import and extend the base class
find src/agents -name '*.ts' -type f | xargs grep -l 'extends AbstractAgent' | head -20

# Locate the project's static analysis or linting configuration and execute the type checker to verify inheritance contracts are satisfied
npx tsc --noEmit

# Identify the test suite for the agents module and run tests to confirm polymorphic behavior works correctly across all agent implementations
npm test -- src/agents
```

**Accept when:**
- All agent implementation files in the agents module contain import statements referencing the base class using relative paths (e.g., `import { AbstractAgent } from './AbstractAgent'` or `from '../AbstractAgent'`) and use `extends` syntax
- TypeScript type checking passes without errors related to missing abstract method implementations or contract violations
- Agent module tests demonstrate that all concrete agent types can be instantiated and used polymorphically through the common interface

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compilation failure or test failures indicate rule violation and MUST be resolved before proceeding.
</enforcement>
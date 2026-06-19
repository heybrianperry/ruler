# Standardize Agent Public Contract Interfaces with IAgent Base Type: Configuration Loading Logic

These rules are ALWAYS ACTIVE for all TypeScript classes in src/agents/ directory that represent agent implementations, utility modules that operate on agents (agent-utils.ts, capabilities.ts, config-utils.ts), configuration loading and validation logic in UnifiedConfigLoader.ts, and MCP capability detection and filtering functions.

### Rules

- **R-AGENT-001** MUST: Configuration loading logic MUST validate input through JSON.parse or parseTOML before processing agent configuration.

### Verify

```bash
# Verify agent implementations use IAgent interface or AbstractAgent base class
grep -r 'implements IAgent\|extends AbstractAgent\|extends AgentsMdAgent' src/agents/*.ts | wc -l

# Verify all agent exports conform to interface pattern
grep -r 'export class.*Agent' src/agents/*.ts | grep -v 'IAgent\|AbstractAgent' | wc -l

# Verify configuration validation patterns are in place
grep -r 'JSON.parse\|parseTOML' src/agents/*.ts src/core/UnifiedConfigLoader.ts | wc -l
```

**Accept when:**
- All agent implementation files in src/agents/ export a class that implements IAgent or extends AbstractAgent/AgentsMdAgent
- Agent utility functions (agent-utils.ts, capabilities.ts, config-utils.ts) accept IAgent interface types in their function signatures
- Configuration loading code uses JSON.parse or parseTOML with validation before constructing agent instances

<enforcement>
Clause R-AGENT-001 is enforced by TypeScript compiler type checking at build time. Code review must verify new agent implementations extend AbstractAgent or implement IAgent. Integration tests must validate that agent instances satisfy the full IAgent contract. Violations result in TypeScript compilation failure or code review rejection. Runtime validation in AbstractAgent constructor throws errors for incomplete implementations. Exceptions require architecture review, technical justification, and engineering lead approval.
</enforcement>
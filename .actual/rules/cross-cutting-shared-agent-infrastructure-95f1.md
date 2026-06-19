# Standardize Agent Public Contract Interfaces with IAgent Base Type: Shared Agent Infrastructure

These rules are ALWAYS ACTIVE for all TypeScript classes in src/agents/ directory that represent agent implementations, utility modules that operate on agents (agent-utils.ts, capabilities.ts, config-utils.ts), configuration loading and validation logic in UnifiedConfigLoader.ts, and MCP capability detection and filtering functions.

### Rules

- **R-AGENT-001** SHOULD: Shared agent infrastructure (capabilities, config-utils, agent-utils) SHOULD depend only on IAgent interface, not concrete implementations.
- **R-AGENT-002** MUST: All agent implementation files in src/agents/ MUST export a class that implements IAgent or extends AbstractAgent/AgentsMdAgent.
- **R-AGENT-003** MUST: Agent utility functions (agent-utils.ts, capabilities.ts, config-utils.ts) MUST accept IAgent interface types in their function signatures.
- **R-AGENT-004** MUST: Configuration loading code MUST use JSON.parse or parseTOML with validation before constructing agent instances.
- **R-AGENT-005** SHOULD: New agent implementations SHOULD extend AbstractAgent or AgentsMdAgent rather than implementing IAgent directly to inherit common functionality.
- **R-AGENT-006** SHOULD: Agent utility functions SHOULD type parameters as IAgent and use type guards when agent-specific behavior is required.

### Verify

```bash
# Verify agent implementations conform to IAgent contract
grep -r 'implements IAgent\|extends AbstractAgent\|extends AgentsMdAgent' src/agents/*.ts | wc -l

# Verify no agent implementations bypass the contract
grep -r 'export class.*Agent' src/agents/*.ts | grep -v 'IAgent\|AbstractAgent' | wc -l

# Verify configuration validation patterns are in use
grep -r 'JSON.parse\|parseTOML' src/agents/*.ts src/core/UnifiedConfigLoader.ts | wc -l
```

**Accept when:**
- All agent implementation files in src/agents/ export a class that implements IAgent or extends AbstractAgent/AgentsMdAgent
- Agent utility functions (agent-utils.ts, capabilities.ts, config-utils.ts) accept IAgent interface types in their function signatures
- Configuration loading code uses JSON.parse or parseTOML with validation before constructing agent instances
- TypeScript compilation succeeds without interface compliance errors
- Integration tests validate that agent instances satisfy the full IAgent contract

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler type checking enforces IAgent interface compliance at build time. Code review MUST verify new agent implementations extend AbstractAgent or implement IAgent. Runtime validation in AbstractAgent constructor MUST throw errors for incomplete implementations. Violations result in compilation failure or code review rejection.
</enforcement>
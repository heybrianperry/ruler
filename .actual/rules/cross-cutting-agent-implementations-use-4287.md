# Standardize Agent Public Contract Interfaces with IAgent Base Type: Agent Implementations Use

These rules are ALWAYS ACTIVE for all TypeScript classes in src/agents/ directory that represent agent implementations, utility modules that operate on agents, configuration loading and validation logic, and MCP capability detection and filtering functions.

### Rules

- **R-AGENT-001** SHOULD: Agent implementations SHOULD use applyRulerConfig for consistent configuration application patterns where applicable.

### Verify

```bash
# Verify agent implementations use IAgent or AbstractAgent
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

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler type checking enforces IAgent interface compliance at build time. Code review process verifies new agent implementations extend AbstractAgent or implement IAgent. Integration tests validate that agent instances satisfy the full IAgent contract.
</enforcement>
# Standardize Agent Public Contract Interfaces with IAgent Base Type: Agent Implementations Export

These rules are ALWAYS ACTIVE for all TypeScript classes in src/agents/ directory that represent agent implementations, utility modules that operate on agents (agent-utils.ts, capabilities.ts, config-utils.ts), configuration loading and validation logic in UnifiedConfigLoader.ts, and MCP capability detection and filtering functions.

### Rules

- **R-AGENT-001** MUST: Agent implementations MUST export their class as the primary public contract (e.g., OpenCodeAgent, MistralVibeAgent, CodexCliAgent).
- **R-AGENT-002** MUST: New agent implementations should extend AbstractAgent or AgentsMdAgent rather than implementing IAgent directly to inherit common functionality.
- **R-AGENT-003** MUST: Use UnifiedConfigLoader.loadUnifiedConfig for configuration loading to ensure consistent validation and parsing patterns.
- **R-AGENT-004** MUST: Agent utility functions should type parameters as IAgent and use type guards when agent-specific behavior is required.
- **R-AGENT-005** MUST: Configuration validation should occur immediately after parsing (JSON.parse/parseTOML) and before passing to agent constructors.

### Verify

```bash
# Verify agent implementations extend AbstractAgent or implement IAgent
grep -r 'implements IAgent\|extends AbstractAgent\|extends AgentsMdAgent' src/agents/*.ts | wc -l

# Verify all agent classes are exported
grep -r 'export class.*Agent' src/agents/*.ts | grep -v 'IAgent\|AbstractAgent' | wc -l

# Verify configuration parsing uses validation patterns
grep -r 'JSON.parse\|parseTOML' src/agents/*.ts src/core/UnifiedConfigLoader.ts | wc -l
```

**Accept when:**
- All agent implementation files in src/agents/ export a class that implements IAgent or extends AbstractAgent/AgentsMdAgent
- Agent utility functions (agent-utils.ts, capabilities.ts, config-utils.ts) accept IAgent interface types in their function signatures
- Configuration loading code uses JSON.parse or parseTOML with validation before constructing agent instances
- TypeScript compilation succeeds with no interface compliance errors
- All agent instances satisfy the full IAgent contract at runtime

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler type checking enforces IAgent interface compliance at build time. Code review process verifies new agent implementations extend AbstractAgent or implement IAgent. Integration tests validate that agent instances satisfy the full IAgent contract. Violation handling includes TypeScript compilation failure, code review rejection, and runtime validation errors in AbstractAgent constructor.
</enforcement>
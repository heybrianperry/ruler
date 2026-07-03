# Standardize Agent Public Contract Interfaces with IAgent Base Type: Agent Implementations Implement

These rules are ALWAYS ACTIVE for all agent implementations and agent-related utilities within the codebase, specifically all TypeScript classes in src/agents/ directory that represent agent implementations, utility modules that operate on agents, configuration loading and validation logic, and MCP capability detection and filtering functions.

### Rules

- **R-AGENT-001** MUST: All agent implementations MUST implement the IAgent interface or extend AbstractAgent base class to ensure contract compliance.
- **R-AGENT-002** MUST: New agent implementations should extend AbstractAgent or AgentsMdAgent rather than implementing IAgent directly to inherit common functionality.
- **R-AGENT-003** MUST: Use UnifiedConfigLoader.loadUnifiedConfig for configuration loading to ensure consistent validation and parsing patterns.
- **R-AGENT-004** MUST: Agent utility functions should type parameters as IAgent and use type guards when agent-specific behavior is required.
- **R-AGENT-005** MUST: Configuration validation should occur immediately after parsing (JSON.parse/parseTOML) and before passing to agent constructors.

### Verify

```bash
# Verify all agent implementations extend AbstractAgent or implement IAgent
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

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler type checking enforces IAgent interface compliance at build time. Code review process verifies new agent implementations extend AbstractAgent or implement IAgent. Integration tests validate that agent instances satisfy the full IAgent contract. Violation handling includes TypeScript compilation failure, code review rejection, and runtime validation errors in AbstractAgent constructor.
</enforcement>
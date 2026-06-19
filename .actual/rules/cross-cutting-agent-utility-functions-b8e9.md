# Standardize Agent Implementation Using Core Interface Contracts and Configuration Loaders: Agent Utility Functions

These rules are ALWAYS ACTIVE for all agent implementations and configuration loading subsystems within the codebase, including all files in src/agents/, configuration loading modules, core interface contracts, MCP capability coordination modules, and agent utility functions.

### Rules

- **R-AGENT-001** SHOULD: Agent utility functions (getAgentOutputPaths, mapRawAgentConfigs) SHOULD be used for common operations rather than implementing agent-specific variants.

### Verify

```bash
# Verify agent implementations use core interface contracts
grep -r 'implements IAgent\|extends AbstractAgent\|extends AgentsMdAgent' src/agents/ | wc -l

# Verify UnifiedConfigLoader is imported in agent and core modules
grep -r 'import.*UnifiedConfigLoader' src/agents/ src/core/ | grep -v test

# Verify consistent parsing patterns across agents
grep -r 'JSON\.parse\|parseTOML' src/agents/ src/core/ | grep -E '(OpenCodeAgent|MistralVibeAgent|CodexCliAgent|AmazonQCliAgent|UnifiedConfigLoader)'

# Verify agent classes are exported as named exports
grep -r 'export class.*Agent' src/agents/ | grep -v test
```

**Accept when:**
- All agent implementation files in src/agents/ extend or implement one of the core interface contracts (IAgent, AbstractAgent, AgentsMdAgent)
- Configuration loading operations use UnifiedConfigLoader with JSON.parse or parseTOML for validation, with no custom parsing logic duplicated across agents
- Agent classes are exported as named exports matching established naming patterns and can be imported by integration modules
- Common operations like getAgentOutputPaths and mapRawAgentConfigs are imported from shared agent-utils rather than reimplemented locally

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis tools MUST check for interface implementation compliance. Code review MUST verify UnifiedConfigLoader usage. Integration tests MUST validate agent public API contracts. CI pipeline MUST fail if agent implementations lack required interface contracts or duplicate configuration parsing logic.
</enforcement>
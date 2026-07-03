# Standardize Agent Implementation Using Core Interface Contracts and Configuration Loaders: Input Validation Configuration

These rules are ALWAYS ACTIVE for all agent implementations and configuration loading subsystems within the codebase, including files in `src/agents/`, configuration loading modules, core interface contracts, MCP capability coordination modules, and agent utility functions.

### Rules

- **R-AGENT-001** MUST: Input validation for configuration data MUST occur at parse time using JSON.parse or parseTOML before processing.

### Verify

```bash
# Verify agent implementations use core interface contracts
grep -r 'implements IAgent\|extends AbstractAgent\|extends AgentsMdAgent' src/agents/ | wc -l

# Verify UnifiedConfigLoader is imported in agent and core modules
grep -r 'import.*UnifiedConfigLoader' src/agents/ src/core/ | grep -v test

# Verify JSON.parse and parseTOML are used for validation in agents
grep -r 'JSON\.parse\|parseTOML' src/agents/ src/core/ | grep -E '(OpenCodeAgent|MistralVibeAgent|CodexCliAgent|AmazonQCliAgent|UnifiedConfigLoader)'

# Verify agent classes are exported as named exports
grep -r 'export class.*Agent' src/agents/ | grep -v test
```

**Accept when:**
- All agent implementation files in `src/agents/` extend or implement one of the core interface contracts (IAgent, AbstractAgent, AgentsMdAgent)
- Configuration loading operations use UnifiedConfigLoader with JSON.parse or parseTOML for validation, with no custom parsing logic duplicated across agents
- Agent classes are exported as named exports matching established naming patterns and can be imported by integration modules
- No direct fs operations exist without FileSystemUtils wrapper in agent configuration loading paths

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All agent implementations and configuration loading modules must comply with R-AGENT-001 before merge.
</enforcement>
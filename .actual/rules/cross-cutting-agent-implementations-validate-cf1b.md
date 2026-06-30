# Standardize applyRulerConfig and loadSubagentFile as Concurrency Coordination Points in Agent Initialization: Agent Implementations Validate

These rules are ALWAYS ACTIVE for all agent implementations and subagent loading workflows within the system, including files in src/agents/ (OpenCodeAgent, ZedAgent, AgentsMdAgent), subagent utility modules (SubagentsUtils, SubagentsProcessor), and configuration loading workflows involving fs/promises, js-yaml, and @iarna/toml.

### Rules

- **R-AGENT-INIT-001** SHOULD: Agent implementations SHOULD validate parsed configuration data using security.input_validation patterns before applying configuration through applyRulerConfig or loadSubagentFile coordination points.

### Verify

```bash
# Verify applyRulerConfig usage across agent implementations
grep -r 'applyRulerConfig' src/agents/ | wc -l

# Verify loadSubagentFile usage in coordination contexts
grep -r 'loadSubagentFile' src/core/Subagents*.ts | grep -v 'export' | wc -l

# Verify parsing occurs before coordination points
grep -r 'JSON.parse.*await.*readFile' src/agents/ | wc -l
```

**Accept when:**
- All agent implementations in src/agents/ use applyRulerConfig as a coordination point after configuration parsing
- All subagent loading workflows use loadSubagentFile for file system access and parsing
- No public API contracts are exported before their respective coordination points complete
- Configuration validation using security.input_validation patterns is applied before coordination point invocation

<enforcement>
Clause Code MUST NOT skip or defer verification of coordination point usage and configuration validation in agent implementations. Code review rejection is required for new agent implementations missing coordination points or validation patterns.
</enforcement>
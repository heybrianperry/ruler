# Standardize applyRulerConfig and loadSubagentFile as Concurrency Coordination Points in Agent Initialization: Subagent Loading Workflows

These rules are ALWAYS ACTIVE for all agent implementations and subagent loading workflows within the system, including files in src/agents/ (OpenCodeAgent, ZedAgent, AgentsMdAgent), subagent utility modules (SubagentsUtils, SubagentsProcessor), and configuration loading workflows involving fs/promises, js-yaml, and @iarna/toml.

### Rules

- **R-SUBAGENT-001** MUST: Subagent loading workflows MUST use loadSubagentFile as the designated concurrency coordination point for file system access and parsing operations.
- **R-SUBAGENT-002** MUST: All agent implementations MUST call applyRulerConfig as a synchronization point after configuration parsing and before exposing the agent instance.
- **R-SUBAGENT-003** MUST: Configuration files (JSON, YAML, TOML) MUST be parsed before coordination points are invoked.
- **R-SUBAGENT-004** MUST: Public API contracts MUST not be exported before their respective coordination points complete.
- **R-SUBAGENT-005** SHOULD: Agent implementations SHOULD add TypeScript return type annotations to coordination functions to make initialization sequencing explicit and type-safe.
- **R-SUBAGENT-006** SHOULD: Agent classes SHOULD track initialization state (e.g., isInitialized flag) to prevent premature use before coordination points complete.

### Verify

```bash
# Verify applyRulerConfig usage across agent implementations
grep -r 'applyRulerConfig' src/agents/ | wc -l

# Verify loadSubagentFile usage in coordination contexts
grep -r 'loadSubagentFile' src/core/Subagents*.ts | grep -v 'export' | wc -l

# Verify parsing occurs before coordination
grep -r 'JSON.parse.*await.*readFile' src/agents/ | wc -l

# Verify no public exports before coordination points
grep -r 'export.*new.*Agent' src/agents/ | grep -v 'applyRulerConfig' | wc -l
```

**Accept when:**
- All agent implementations in src/agents/ use applyRulerConfig as a coordination point after configuration parsing
- All subagent loading workflows use loadSubagentFile for file system access and parsing
- No public API contracts are exported before their respective coordination points complete
- Configuration files are parsed before coordination points are invoked
- TypeScript return type annotations are present on coordination functions

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All new agent implementations and subagent loading workflows must satisfy the coordination point requirements before code review approval.
</enforcement>
# Standardize applyRulerConfig and loadSubagentFile as Concurrency Coordination Points in Agent Initialization: Agent Implementations Introduce

These rules are ALWAYS ACTIVE for all agent implementations and subagent loading workflows within the system, including all files in src/agents/ (OpenCodeAgent, ZedAgent, AgentsMdAgent), subagent utility modules (SubagentsUtils, SubagentsProcessor), and configuration loading workflows involving fs/promises, js-yaml, and @iarna/toml.

### Rules

- **R-AGENT-INIT-001** MAY: Agent implementations MAY introduce additional coordination points for domain-specific initialization requirements beyond applyRulerConfig.
- **R-AGENT-INIT-002** MUST: All agent implementations in src/agents/ use applyRulerConfig as a coordination point after configuration parsing and before exposing the agent instance.
- **R-AGENT-INIT-003** MUST: All subagent loading workflows use loadSubagentFile from SubagentsUtils.ts as the single entry point for file access, ensuring consistent error handling and validation.
- **R-AGENT-INIT-004** SHOULD: Agent implementations follow the pattern in OpenCodeAgent.ts and ZedAgent.ts: parse configuration files first, then call applyRulerConfig before exposing the agent instance.
- **R-AGENT-INIT-005** SHOULD: Add TypeScript return type annotations to coordination functions to make initialization sequencing explicit and type-safe.
- **R-AGENT-INIT-006** SHOULD: Consider adding initialization state tracking (e.g., isInitialized flag) to agent classes to prevent premature use before coordination points complete.

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
- Configuration files (JSON, YAML, TOML) are validated and parsed before agent instantiation
- TypeScript return type annotations are present on coordination functions

<enforcement>
Code review MUST verify coordination point usage in new agent implementations. Static analysis rules MUST detect async file operations not coordinated through designated functions. Integration tests MUST validate initialization sequencing and configuration application. Claude Code MUST NOT skip or defer verification of these rules.
</enforcement>
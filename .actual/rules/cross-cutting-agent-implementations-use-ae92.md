# Standardize applyRulerConfig and loadSubagentFile as Concurrency Coordination Points in Agent Initialization: Agent Implementations Use

These rules are ALWAYS ACTIVE for all agent implementations and subagent loading workflows within the system, including files in src/agents/ (OpenCodeAgent, ZedAgent, AgentsMdAgent), subagent utility modules (SubagentsUtils, SubagentsProcessor), and configuration loading workflows involving fs/promises, js-yaml, and @iarna/toml.

### Rules

- **R-AGENT-INIT-001** MUST: Agent implementations MUST use applyRulerConfig as the designated concurrency coordination point for applying rule-based configuration during initialization.
- **R-AGENT-INIT-002** MUST: All agent implementations in src/agents/ MUST parse configuration files before calling applyRulerConfig.
- **R-AGENT-INIT-003** MUST: Subagent loading workflows MUST use loadSubagentFile from SubagentsUtils.ts as the single entry point for file access, ensuring consistent error handling and validation.
- **R-AGENT-INIT-004** SHOULD: Agent implementations SHOULD add TypeScript return type annotations to coordination functions to make initialization sequencing explicit and type-safe.
- **R-AGENT-INIT-005** SHOULD: Agent classes SHOULD implement initialization state tracking (e.g., isInitialized flag) to prevent premature use before coordination points complete.

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
- Configuration files are parsed before coordination points are invoked
- Initialization sequencing is validated in integration tests

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules during code review or implementation of new agent types. Violations require code review rejection or refactoring of existing agents during maintenance.
</enforcement>
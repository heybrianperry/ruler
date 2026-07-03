# Standardize applyRulerConfig and loadSubagentFile as Concurrency Coordination Points in Agent Initialization: Subagent Discovery Functions

These rules are ALWAYS ACTIVE for all agent implementations and subagent loading workflows within the system, including files in src/agents/ (OpenCodeAgent, ZedAgent, AgentsMdAgent), subagent utility modules (SubagentsUtils, SubagentsProcessor), and configuration loading workflows involving fs/promises, js-yaml, and @iarna/toml.

### Rules

- **R-CONC-001** SHOULD: Subagent discovery functions (discoverSubagents, listMarkdownFilesRecursive) SHOULD coordinate file system traversal through consistent async/await patterns.
- **R-CONC-002** SHOULD: Agent implementations SHOULD use applyRulerConfig as a synchronization point for rule-based configuration after parsing configuration files.
- **R-CONC-003** SHOULD: Subagent loading workflows SHOULD use loadSubagentFile from SubagentsUtils.ts as the single entry point for file access, ensuring consistent error handling and validation.
- **R-CONC-004** SHOULD: New agent implementations SHOULD follow the pattern in OpenCodeAgent.ts and ZedAgent.ts: parse configuration files first, then call applyRulerConfig before exposing the agent instance.
- **R-CONC-005** SHOULD: Agent classes SHOULD include TypeScript return type annotations on coordination functions to make initialization sequencing explicit and type-safe.
- **R-CONC-006** MAY: Agent implementations MAY add initialization state tracking (e.g., isInitialized flag) to prevent premature use before coordination points complete.

### Verify

```bash
# Verify applyRulerConfig usage across agent implementations
grep -r 'applyRulerConfig' src/agents/ | wc -l

# Verify loadSubagentFile usage in coordination contexts
grep -r 'loadSubagentFile' src/core/Subagents*.ts | grep -v 'export' | wc -l

# Verify parsing before coordination points
grep -r 'JSON.parse.*await.*readFile' src/agents/ | wc -l
```

**Accept when:**
- All agent implementations in src/agents/ use applyRulerConfig as a coordination point after configuration parsing
- All subagent loading workflows use loadSubagentFile for file system access and parsing
- No public API contracts are exported before their respective coordination points complete
- Configuration files (JSON, YAML, TOML) are parsed before coordination points are invoked
- Async/await patterns are consistently applied across all file system traversal operations

<enforcement>
Code review MUST verify coordination point usage in new agent implementations. Static analysis rules MUST detect async file operations not coordinated through designated functions. Integration tests MUST validate initialization sequencing and configuration application. Claude Code MUST NOT skip or defer verification of these rules.
</enforcement>
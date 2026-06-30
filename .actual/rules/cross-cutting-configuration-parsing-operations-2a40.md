# Standardize applyRulerConfig and loadSubagentFile as Concurrency Coordination Points in Agent Initialization: Configuration Parsing Operations

These rules are ALWAYS ACTIVE for all agent implementations and subagent loading workflows within the system, including all files in src/agents/ (OpenCodeAgent, ZedAgent, AgentsMdAgent), subagent utility modules (SubagentsUtils, SubagentsProcessor), and configuration loading workflows involving fs/promises, js-yaml, and @iarna/toml.

### Rules

- **R-CONC-001** MUST: Configuration parsing operations (JSON.parse, YAML parsing) MUST complete before applyRulerConfig or loadSubagentFile coordination points are invoked.

### Verify

```bash
# Verify applyRulerConfig usage across agent implementations
grep -r 'applyRulerConfig' src/agents/ | wc -l

# Verify loadSubagentFile usage in coordination contexts
grep -r 'loadSubagentFile' src/core/Subagents*.ts | grep -v 'export' | wc -l

# Verify parsing operations precede coordination points
grep -r 'JSON.parse.*await.*readFile' src/agents/ | wc -l
```

**Accept when:**
- All agent implementations in src/agents/ use applyRulerConfig as a coordination point after configuration parsing
- All subagent loading workflows use loadSubagentFile for file system access and parsing
- No public API contracts are exported before their respective coordination points complete
- Configuration parsing (JSON.parse, YAML parsing) operations complete before applyRulerConfig or loadSubagentFile are called

<enforcement>
Clause Code MUST NOT skip or defer verification of coordination point sequencing in agent initialization workflows. Violations must be caught during code review and require refactoring before merge.
</enforcement>
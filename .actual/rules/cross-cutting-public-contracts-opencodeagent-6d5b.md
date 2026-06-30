# Standardize applyRulerConfig and loadSubagentFile as Concurrency Coordination Points in Agent Initialization: Public Contracts Opencodeagent

These rules are ALWAYS ACTIVE for all agent implementations and subagent loading workflows within the system, including all files in src/agents/ (OpenCodeAgent, ZedAgent, AgentsMdAgent), subagent utility modules (SubagentsUtils, SubagentsProcessor), and configuration loading workflows involving fs/promises, js-yaml, and @iarna/toml.

### Rules

- **R-CONC-001** MUST: Public API contracts (OpenCodeAgent, ZedAgent, ParsedFrontmatter, CopilotToolMapping) MUST NOT be exposed until their respective coordination points have completed.
- **R-CONC-002** MUST: All agent implementations in src/agents/ use applyRulerConfig as a coordination point after configuration parsing and before exposing the agent instance.
- **R-CONC-003** MUST: All subagent loading workflows use loadSubagentFile from SubagentsUtils.ts as the single entry point for file access, ensuring consistent error handling and validation.
- **R-CONC-004** SHOULD: Add TypeScript return type annotations to coordination functions to make initialization sequencing explicit and type-safe.
- **R-CONC-005** SHOULD: Consider adding initialization state tracking (e.g., isInitialized flag) to agent classes to prevent premature use before coordination points complete.

### Verify

```bash
# Verify applyRulerConfig usage across agent implementations
grep -r 'applyRulerConfig' src/agents/ | wc -l

# Verify loadSubagentFile usage in coordination contexts
grep -r 'loadSubagentFile' src/core/Subagents*.ts | grep -v 'export' | wc -l

# Verify parsing occurs before coordination points
grep -r 'JSON.parse.*await.*readFile' src/agents/ | wc -l

# Verify no public API contracts exported before coordination
grep -r 'export.*OpenCodeAgent\|export.*ZedAgent\|export.*ParsedFrontmatter\|export.*CopilotToolMapping' src/agents/ src/core/
```

**Accept when:**
- All agent implementations in src/agents/ use applyRulerConfig as a coordination point after configuration parsing
- All subagent loading workflows use loadSubagentFile for file system access and parsing
- No public API contracts are exported before their respective coordination points complete
- TypeScript return type annotations are present on coordination functions
- Initialization state tracking is implemented in agent classes

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review checklist MUST require verification of coordination point usage in new agent implementations. Static analysis rules MUST detect async file operations not coordinated through designated functions. Integration tests MUST validate initialization sequencing and configuration application.
</enforcement>
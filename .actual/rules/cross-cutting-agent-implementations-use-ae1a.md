# Standardize applyRulerConfig as Configuration Application Pattern: Agent Implementations Use

These rules are ALWAYS ACTIVE for all agent implementations and configuration loading modules within the Ruler system, including all files in src/agents/ directory, configuration loading modules (UnifiedConfigLoader, ConfigLoader), reversion and cleanup operations in revert-engine.ts, and agent utility functions that manipulate configuration state.

### Rules

- **R-AGENT-001** MUST: Agent implementations MUST use core library imports from './IAgent' interface or './AgentsMdAgent' base class to ensure contract compliance.
- **R-AGENT-002** MUST: All agent implementation files in src/agents/ MUST contain applyRulerConfig method or function definition as the primary configuration entry point.
- **R-AGENT-003** MUST: Configuration parsing operations (JSON.parse for JSON files, parseTOML from @iarna/toml for TOML files) MUST be wrapped in try-catch blocks with meaningful error messages.
- **R-AGENT-004** MUST: File system operations MUST use FileSystemUtils or fs/promises to maintain compatibility with revert-engine backup and restore operations.
- **R-AGENT-005** MUST: Agent implementations MUST coordinate with UnifiedConfigLoader by accepting configuration objects that have already been processed through hash computation and rule processing pipelines.
- **R-AGENT-006** SHOULD: Agent-specific configuration extensions SHOULD be documented in module headers and SHOULD NOT break the core applyRulerConfig contract.

### Verify

```bash
# Count applyRulerConfig occurrences in agent implementations
grep -r 'applyRulerConfig' src/agents/ --include='*.ts' | wc -l

# Verify configuration parsing is present in applyRulerConfig implementations
grep -r 'JSON\.parse\|parseTOML' src/agents/ --include='*.ts' -A 2 | grep -c 'applyRulerConfig'

# Verify IAgent interface imports in agent implementations
grep -r "from.*['\"]\./(IAgent|AgentsMdAgent)" src/agents/ --include='*.ts' | wc -l
```

**Accept when:**
- All agent implementation files in src/agents/ contain applyRulerConfig method or function definition
- Configuration parsing operations (JSON.parse, parseTOML) are present in agent files that implement applyRulerConfig
- Agent implementations import from './IAgent' or './AgentsMdAgent' to maintain interface contract compliance
- File system operations use FileSystemUtils or fs/promises for revert-engine compatibility

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST fail if new agent implementations lack applyRulerConfig method. Code review MUST block merge if agent does not import from IAgent or AgentsMdAgent base classes. Integration tests MUST validate configuration application and reversion workflows for each agent type.
</enforcement>
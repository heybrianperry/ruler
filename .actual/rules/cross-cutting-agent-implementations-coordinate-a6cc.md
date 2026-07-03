# Standardize applyRulerConfig as Configuration Application Pattern: Agent Implementations Coordinate

These rules are ALWAYS ACTIVE for all agent implementations and configuration loading modules within the Ruler system, including all files in src/agents/ directory, configuration loading modules (UnifiedConfigLoader, ConfigLoader), reversion and cleanup operations in revert-engine.ts, and agent utility functions that manipulate configuration state.

### Rules

- **R-AGENT-001** SHOULD: Agent implementations SHOULD coordinate with UnifiedConfigLoader for configuration loading to benefit from centralized hash computation and rule processing.
- **R-AGENT-002** MUST: New agent implementations MUST extend IAgent interface or AgentsMdAgent base class and implement applyRulerConfig as the primary configuration entry point.
- **R-AGENT-003** MUST: Configuration parsing operations MUST use JSON.parse for JSON configuration files and parseTOML from @iarna/toml for TOML files, wrapping parse operations in try-catch blocks with meaningful error messages.
- **R-AGENT-004** MUST: File system operations MUST use FileSystemUtils or fs/promises to maintain compatibility with revert-engine backup and restore operations.
- **R-AGENT-005** SHOULD: Agent implementations SHOULD coordinate with UnifiedConfigLoader by accepting configuration objects that have already been processed through hash computation and rule processing pipelines.
- **R-AGENT-006** SHOULD: Agent-specific configuration extensions SHOULD be documented in module headers and SHOULD NOT break the core applyRulerConfig contract.

### Verify

```bash
# Verify applyRulerConfig presence in agent implementations
grep -r 'applyRulerConfig' src/agents/ --include='*.ts' | wc -l

# Verify configuration parsing operations are present in agent files
grep -r 'JSON\.parse\|parseTOML' src/agents/ --include='*.ts' -A 2 | grep -c 'applyRulerConfig'

# Verify IAgent interface imports
grep -r "from.*['\"]\./(IAgent|AgentsMdAgent)" src/agents/ --include='*.ts' | wc -l
```

**Accept when:**
- All agent implementation files in src/agents/ contain applyRulerConfig method or function definition
- Configuration parsing operations (JSON.parse, parseTOML) are present in agent files that implement applyRulerConfig
- Agent implementations import from './IAgent' or './AgentsMdAgent' to maintain interface contract compliance
- New agent implementations extend IAgent interface or AgentsMdAgent base class
- File system operations use FileSystemUtils or fs/promises

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST fail if new agent implementations lack applyRulerConfig method. Code review MUST block merge if agent does not import from IAgent or AgentsMdAgent base classes. Integration tests MUST validate configuration application and reversion workflows for each agent type.
</enforcement>
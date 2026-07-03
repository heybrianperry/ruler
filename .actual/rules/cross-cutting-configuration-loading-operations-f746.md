# Standardize applyRulerConfig as Configuration Application Pattern: Configuration Loading Operations

These rules are ALWAYS ACTIVE for all agent implementations and configuration loading modules within the Ruler system, including all files in src/agents/ directory, configuration loading modules (UnifiedConfigLoader, ConfigLoader), reversion and cleanup operations in revert-engine.ts, and agent utility functions that manipulate configuration state.

### Rules

- **R-CONFIG-001** MUST: Configuration loading operations MUST use FileSystemUtils or fs/promises for file system access to maintain consistency with revert-engine expectations.
- **R-CONFIG-002** MUST: All agent implementation files in src/agents/ MUST contain applyRulerConfig method or function definition.
- **R-CONFIG-003** MUST: Configuration parsing operations (JSON.parse, parseTOML) MUST be wrapped in try-catch blocks with meaningful error messages.
- **R-CONFIG-004** MUST: Agent implementations MUST import from './IAgent' or './AgentsMdAgent' to maintain interface contract compliance.
- **R-CONFIG-005** SHOULD: New agent implementations should extend IAgent interface or AgentsMdAgent base class and implement applyRulerConfig as the primary configuration entry point.
- **R-CONFIG-006** SHOULD: Use JSON.parse for JSON configuration files and parseTOML from @iarna/toml for TOML files.
- **R-CONFIG-007** SHOULD: Coordinate with UnifiedConfigLoader by accepting configuration objects that have already been processed through hash computation and rule processing pipelines.

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
- File system operations in configuration loading use FileSystemUtils or fs/promises
- Configuration parsing is wrapped in try-catch blocks with meaningful error messages

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST fail if new agent implementations lack applyRulerConfig method. Code review MUST block merge if agent does not import from IAgent or AgentsMdAgent base classes. Integration tests MUST validate configuration application and reversion workflows for each agent type.
</enforcement>
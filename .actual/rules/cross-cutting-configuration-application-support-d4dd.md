# Standardize applyRulerConfig as Configuration Application Pattern: Configuration Application Support

These rules are ALWAYS ACTIVE for all agent implementations and configuration loading modules within the Ruler system, including all files in src/agents/, configuration loading modules (UnifiedConfigLoader, ConfigLoader), reversion operations in revert-engine.ts, and agent utility functions that manipulate configuration state.

### Rules

- **R-CONFIG-001** SHOULD: Configuration application SHOULD support both JSON and TOML formats using @iarna/toml for TOML parsing where applicable.
- **R-CONFIG-002** MUST: All agent implementation files in src/agents/ MUST contain applyRulerConfig method or function definition.
- **R-CONFIG-003** MUST: Configuration parsing operations (JSON.parse, parseTOML) MUST be wrapped in try-catch blocks with meaningful error messages.
- **R-CONFIG-004** MUST: Agent implementations MUST import from './IAgent' or './AgentsMdAgent' to maintain interface contract compliance.
- **R-CONFIG-005** MUST: File system operations MUST use FileSystemUtils or fs/promises to maintain compatibility with revert-engine backup and restore operations.
- **R-CONFIG-006** SHOULD: New agent implementations SHOULD extend IAgent interface or AgentsMdAgent base class and implement applyRulerConfig as the primary configuration entry point.
- **R-CONFIG-007** SHOULD: Agent-specific configuration extensions SHOULD be documented in module headers and SHOULD NOT break the core applyRulerConfig contract.

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
- All configuration parsing operations are wrapped in try-catch blocks with meaningful error messages
- File system operations use FileSystemUtils or fs/promises

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST fail if new agent implementations lack applyRulerConfig method. Code review MUST block merge if agent does not import from IAgent or AgentsMdAgent base classes. Runtime warnings MUST be logged when configuration application bypasses standard validation patterns.
</enforcement>
# Standardize applyRulerConfig as Configuration Application Pattern: Agent Implementations Implement

These rules are ALWAYS ACTIVE for all agent implementations and configuration loading modules within the Ruler system, specifically all files in `src/agents/` directory and configuration loading modules that integrate with the Ruler configuration system.

### Rules

- **R-AGENT-001** MUST: All agent implementations MUST implement an `applyRulerConfig` method or function that serves as the primary entry point for configuration application.
- **R-AGENT-002** MUST: Configuration parsing operations (JSON.parse for JSON files, parseTOML from @iarna/toml for TOML files) MUST be wrapped in try-catch blocks with meaningful error messages.
- **R-AGENT-003** MUST: Agent implementations MUST extend IAgent interface or AgentsMdAgent base class to maintain interface contract compliance.
- **R-AGENT-004** MUST: File system operations MUST use FileSystemUtils or fs/promises to maintain compatibility with revert-engine backup and restore operations.
- **R-AGENT-005** SHOULD: Agent-specific configuration extensions SHOULD be documented in module headers and SHOULD NOT break the core applyRulerConfig contract.
- **R-AGENT-006** SHOULD: New agent implementations SHOULD coordinate with UnifiedConfigLoader by accepting configuration objects that have already been processed through hash computation and rule processing pipelines.

### Verify

```bash
# Count applyRulerConfig implementations in agent files
grep -r 'applyRulerConfig' src/agents/ --include='*.ts' | wc -l

# Verify configuration parsing is present in applyRulerConfig implementations
grep -r 'JSON\.parse\|parseTOML' src/agents/ --include='*.ts' -A 2 | grep -c 'applyRulerConfig'

# Verify IAgent interface imports in agent implementations
grep -r "from.*['\"]\./(IAgent|AgentsMdAgent)" src/agents/ --include='*.ts' | wc -l
```

**Accept when:**
- All agent implementation files in `src/agents/` contain `applyRulerConfig` method or function definition
- Configuration parsing operations (JSON.parse, parseTOML) are present in agent files that implement `applyRulerConfig`
- Agent implementations import from `./IAgent` or `./AgentsMdAgent` to maintain interface contract compliance
- All configuration parsing is wrapped in try-catch blocks with error handling
- File system operations use FileSystemUtils or fs/promises

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. CI pipeline MUST fail if new agent implementations lack `applyRulerConfig` method. Code review MUST block merge if agent does not import from IAgent or AgentsMdAgent base classes.
</enforcement>
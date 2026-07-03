# Standardize Agent Implementation Using Core Interface Contracts and Configuration Loaders: Agent Implementations Import

These rules are ALWAYS ACTIVE for all agent implementations and configuration loading subsystems within the codebase, specifically targeting files in `src/agents/` directory and core configuration modules.

### Rules

- **R-AGENT-001** MUST: Agent implementations MUST import core dependencies (path, fs, fs/promises) for file system operations rather than implementing custom file handling.
- **R-AGENT-002** MUST: Agent implementations MUST extend or implement one of the core interface contracts (IAgent, AbstractAgent, AgentsMdAgent).
- **R-AGENT-003** MUST: Configuration loading operations MUST use UnifiedConfigLoader with JSON.parse or parseTOML for validation, with no custom parsing logic duplicated across agents.
- **R-AGENT-004** MUST: Agent classes MUST be exported as named exports matching the agent name pattern (e.g., export class OpenCodeAgent).
- **R-AGENT-005** SHOULD: New agent implementations should extend AbstractAgent or AgentsMdAgent rather than implementing IAgent directly unless specific requirements dictate otherwise.
- **R-AGENT-006** SHOULD: Use UnifiedConfigLoader.loadUnifiedConfig with appropriate format hints (TOML vs JSON) based on configuration file extension.
- **R-AGENT-007** SHOULD: Import file system utilities from core/FileSystemUtils rather than using fs/fs.promises directly to benefit from error handling and path normalization.
- **R-AGENT-008** SHOULD: For agents requiring MCP support, call agentSupportsMcp early in initialization to determine capability availability before attempting MCP operations.

### Verify

```bash
# Verify interface implementation compliance
grep -r 'implements IAgent\|extends AbstractAgent\|extends AgentsMdAgent' src/agents/ | wc -l

# Verify UnifiedConfigLoader usage
grep -r 'import.*UnifiedConfigLoader' src/agents/ src/core/ | grep -v test

# Verify standardized parsing patterns
grep -r 'JSON\.parse\|parseTOML' src/agents/ src/core/ | grep -E '(OpenCodeAgent|MistralVibeAgent|CodexCliAgent|AmazonQCliAgent|UnifiedConfigLoader)'

# Verify named export patterns
grep -r 'export class.*Agent' src/agents/ | grep -v test
```

**Accept when:**
- All agent implementation files in src/agents/ extend or implement one of the core interface contracts (IAgent, AbstractAgent, AgentsMdAgent)
- Configuration loading operations use UnifiedConfigLoader with JSON.parse or parseTOML for validation, with no custom parsing logic duplicated across agents
- Agent classes are exported as named exports matching established naming patterns and can be imported by integration modules
- Core dependencies (path, fs, fs/promises) are imported for file system operations rather than custom implementations

<enforcement>
Claude Code MUST NOT skip or defer verification. All agent implementations must pass the verify commands before acceptance. CI pipeline must fail if agent implementations lack required interface contracts or use custom file handling instead of core dependencies.
</enforcement>
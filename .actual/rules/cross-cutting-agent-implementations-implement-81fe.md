# Standardize Agent Implementation Using Core Interface Contracts and Configuration Loaders: Agent Implementations Implement

These rules are ALWAYS ACTIVE for all agent implementations and configuration loading subsystems within the codebase, specifically targeting files in `src/agents/` directory and core configuration modules.

### Rules

- **R-AGENT-001** MUST: All agent implementations MUST implement or extend one of the core interface contracts: IAgent, AbstractAgent, or AgentsMdAgent.
- **R-AGENT-002** MUST: All configuration loading operations MUST use UnifiedConfigLoader with JSON.parse or parseTOML for validation, with no custom parsing logic duplicated across agents.
- **R-AGENT-003** MUST: Agent classes MUST be exported as named exports matching the agent name pattern (e.g., `export class OpenCodeAgent`) to maintain consistent public API discovery.
- **R-AGENT-004** SHOULD: New agent implementations SHOULD extend AbstractAgent or AgentsMdAgent rather than implementing IAgent directly unless specific requirements dictate otherwise.
- **R-AGENT-005** SHOULD: File system operations SHOULD use core/FileSystemUtils rather than using fs/fs.promises directly to benefit from error handling and path normalization.
- **R-AGENT-006** SHOULD: For agents requiring MCP support, call agentSupportsMcp early in initialization to determine capability availability before attempting MCP operations.

### Verify

```bash
# Verify all agent implementations extend or implement core interface contracts
grep -r 'implements IAgent\|extends AbstractAgent\|extends AgentsMdAgent' src/agents/ | wc -l

# Verify UnifiedConfigLoader usage across agents and core modules
grep -r 'import.*UnifiedConfigLoader' src/agents/ src/core/ | grep -v test

# Verify standardized parsing patterns (JSON.parse and parseTOML)
grep -r 'JSON\.parse\|parseTOML' src/agents/ src/core/ | grep -E '(OpenCodeAgent|MistralVibeAgent|CodexCliAgent|AmazonQCliAgent|UnifiedConfigLoader)'

# Verify agent classes are exported as named exports
grep -r 'export class.*Agent' src/agents/ | grep -v test
```

**Accept when:**
- All agent implementation files in `src/agents/` extend or implement one of the core interface contracts (IAgent, AbstractAgent, AgentsMdAgent)
- Configuration loading operations use UnifiedConfigLoader with JSON.parse or parseTOML for validation, with no custom parsing logic duplicated across agents
- Agent classes are exported as named exports matching established naming patterns and can be imported by integration modules
- Dependency graph analysis confirms agents import from approved core modules (FileSystemUtils, UnifiedConfigLoader, mcp/capabilities)

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Static analysis tools checking for interface implementation compliance, code review checklists requiring UnifiedConfigLoader usage, integration tests validating agent public API contracts, and dependency graph analysis are mandatory before accepting agent implementation changes.
</enforcement>
# Standardize Agent Implementation Using Core Interface Contracts and Configuration Loaders: Agents Supporting Mcp

These rules are ALWAYS ACTIVE for all agent implementations and configuration loading subsystems within the codebase, including all files in src/agents/, configuration loading modules, core interface contracts, MCP capability coordination modules, and agent utility functions.

### Rules

- **R-AGENT-001** SHOULD: Agents supporting MCP capabilities SHOULD use getAgentMcpCapabilities, agentSupportsMcp, and filterMcpConfigForAgent from mcp/capabilities module

### Verify

```bash
# Verify agent implementations use core interface contracts
grep -r 'implements IAgent\|extends AbstractAgent\|extends AgentsMdAgent' src/agents/ | wc -l

# Verify UnifiedConfigLoader usage across agents and core modules
grep -r 'import.*UnifiedConfigLoader' src/agents/ src/core/ | grep -v test

# Verify standardized parsing patterns (JSON.parse, parseTOML) in key agents
grep -r 'JSON\.parse\|parseTOML' src/agents/ src/core/ | grep -E '(OpenCodeAgent|MistralVibeAgent|CodexCliAgent|AmazonQCliAgent|UnifiedConfigLoader)'

# Verify agent classes are exported as named exports
grep -r 'export class.*Agent' src/agents/ | grep -v test
```

**Accept when:**
- All agent implementation files in src/agents/ extend or implement one of the core interface contracts (IAgent, AbstractAgent, AgentsMdAgent)
- Configuration loading operations use UnifiedConfigLoader with JSON.parse or parseTOML for validation, with no custom parsing logic duplicated across agents
- Agent classes are exported as named exports matching established naming patterns and can be imported by integration modules
- Agents supporting MCP capabilities import and use functions from mcp/capabilities module

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis tools MUST check for interface implementation compliance in agent files. Code review MUST require UnifiedConfigLoader usage for new agent configurations. Integration tests MUST validate agent public API contracts match expected interfaces. Dependency graph analysis MUST ensure agents import from approved core modules. CI pipeline MUST fail if agent implementations lack required interface contracts. Pull requests MUST be blocked if custom configuration parsing duplicates UnifiedConfigLoader functionality. Architecture review MUST be required for agents not using standard core dependencies. Automated linting rules MUST flag direct fs operations without FileSystemUtils wrapper.
</enforcement>
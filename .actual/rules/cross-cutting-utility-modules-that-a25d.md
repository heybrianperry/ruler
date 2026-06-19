# Standardize Agent Public Contract Exports via IAgent Interface: Utility Modules That

These rules are ALWAYS ACTIVE for all TypeScript modules in `src/agents/` that implement agent behavior, core utility modules that load or transform agent configurations, MCP integration modules that query agent capabilities, and agent utility functions that operate polymorphically on agent instances.

### Rules

- **R-AGENT-001** MUST: Utility modules that operate on agents (agent-utils, capabilities, config-utils) MUST depend on the IAgent interface contract rather than concrete agent implementations.
- **R-AGENT-002** MUST: All agent implementation files in `src/agents/` MUST export a class that implements IAgent or extends AbstractAgent/AgentsMdAgent.
- **R-AGENT-003** MUST: All agent implementations that parse external configuration MUST use standard validation patterns (JSON.parse, parseTOML) with error handling wrapped in try-catch blocks.
- **R-AGENT-004** MUST: Core utility modules (config-utils.ts, capabilities.ts, agent-utils.ts) MUST import and depend on the IAgent interface rather than concrete agent classes.
- **R-AGENT-005** MUST: All agent implementations MUST export the agent class as the default export and ensure it is listed in the api.public.contracts facet.
- **R-AGENT-006** MUST: When implementing applyRulerConfig, the method MUST be async and handle concurrent configuration updates safely to prevent race conditions.
- **R-AGENT-007** SHOULD: New agent implementations should start by importing IAgent from './IAgent' and implementing all required methods before adding agent-specific logic.
- **R-AGENT-008** SHOULD: Use AbstractAgent or AgentsMdAgent base classes when agents share common functionality, ensuring the base class itself implements IAgent.

### Verify

```bash
# Count agent implementations that comply with IAgent interface
grep -r "implements.*IAgent\|extends.*AbstractAgent\|extends.*AgentsMdAgent" src/agents/*.ts | wc -l

# Count agent files with api.public.contracts export
grep -r "api.public.contracts=" src/agents/*.ts | wc -l

# Count IAgent imports across agent, core, and mcp modules
grep -r "import.*IAgent" src/agents/*.ts src/core/*.ts src/mcp/*.ts | wc -l

# Find agent files that lack IAgent/AbstractAgent/AgentsMdAgent compliance
find src/agents -name "*Agent.ts" -exec grep -L "IAgent\|AbstractAgent\|AgentsMdAgent" {} \;
```

**Accept when:**
- All agent implementation files in `src/agents/` export a class that implements IAgent or extends AbstractAgent/AgentsMdAgent
- Core utility modules (config-utils.ts, capabilities.ts, agent-utils.ts) import and depend on the IAgent interface rather than concrete agent classes
- No agent implementation files exist that lack the api.public.contracts export pattern or IAgent interface compliance
- All agent implementations that parse external configuration use standard validation patterns (JSON.parse, parseTOML) with error handling
- TypeScript compiler type checking enforces IAgent interface compliance at build time

<enforcement>
Claude Code MUST NOT skip or defer verification. Build failures occur when agent implementations do not satisfy IAgent interface type constraints. Code review blocks merge requests for new agents that lack proper contract exports or interface compliance. Linting warnings are escalated to errors for agent modules missing IAgent imports or implementations.
</enforcement>
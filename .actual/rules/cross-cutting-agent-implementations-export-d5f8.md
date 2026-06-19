# Standardize Agent Public Contract Exports via IAgent Interface: Agent Implementations Export

These rules are ALWAYS ACTIVE for all TypeScript modules in `src/agents/` that implement agent behavior, core utility modules that load or transform agent configurations, MCP integration modules that query agent capabilities, and agent utility functions that operate polymorphically on agent instances.

### Rules

- **R-AGENT-001** MUST: All agent implementations MUST export a public contract class that implements or extends the IAgent interface or AbstractAgent/AgentsMdAgent base classes.

### Verify

```bash
# Count agent implementations that comply with IAgent interface
grep -r "implements.*IAgent\|extends.*AbstractAgent\|extends.*AgentsMdAgent" src/agents/*.ts | wc -l

# Count agent implementations with api.public.contracts export
grep -r "api.public.contracts=" src/agents/*.ts | wc -l

# Count IAgent imports across agent, core, and mcp modules
grep -r "import.*IAgent" src/agents/*.ts src/core/*.ts src/mcp/*.ts | wc -l

# Find agent implementation files lacking IAgent/AbstractAgent/AgentsMdAgent compliance
find src/agents -name "*Agent.ts" -exec grep -L "IAgent\|AbstractAgent\|AgentsMdAgent" {} \;
```

**Accept when:**
- All agent implementation files in `src/agents/` export a class that implements IAgent or extends AbstractAgent/AgentsMdAgent
- Core utility modules (config-utils.ts, capabilities.ts, agent-utils.ts) import and depend on the IAgent interface rather than concrete agent classes
- No agent implementation files exist that lack the api.public.contracts export pattern or IAgent interface compliance
- All agent implementations that parse external configuration use standard validation patterns (JSON.parse, parseTOML) with error handling

<enforcement>
Clause Code MUST NOT skip or defer verification. TypeScript compiler type checking enforces IAgent interface compliance at build time. Code review blocks merge requests for new agents that lack proper contract exports or interface compliance. Linting warnings are escalated to errors for agent modules missing IAgent imports or implementations.
</enforcement>
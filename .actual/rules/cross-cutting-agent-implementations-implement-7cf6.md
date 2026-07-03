# Standardize Agent Public Contract Exports via IAgent Interface: Agent Implementations Implement

These rules are ALWAYS ACTIVE for all TypeScript modules in `src/agents/` that implement agent behavior, core utility modules that load or transform agent configurations, MCP integration modules that query agent capabilities, and agent utility functions that operate polymorphically on agent instances.

### Rules

- **R-AGENT-001** SHOULD: Agent implementations SHOULD implement the `applyRulerConfig` concurrency model pattern for consistent configuration application behavior.

### Verify

```bash
# Count agent implementations that declare IAgent interface compliance
grep -r "implements.*IAgent\|extends.*AbstractAgent\|extends.*AgentsMdAgent" src/agents/*.ts | wc -l

# Count agent modules exporting public contracts
grep -r "api.public.contracts=" src/agents/*.ts | wc -l

# Count IAgent imports across agent, core, and MCP modules
grep -r "import.*IAgent" src/agents/*.ts src/core/*.ts src/mcp/*.ts | wc -l

# Find agent implementation files lacking IAgent/AbstractAgent/AgentsMdAgent compliance
find src/agents -name "*Agent.ts" -exec grep -L "IAgent\|AbstractAgent\|AgentsMdAgent" {} \;
```

**Accept when:**
- All agent implementation files in `src/agents/` export a class that implements `IAgent` or extends `AbstractAgent`/`AgentsMdAgent`
- Core utility modules (`config-utils.ts`, `capabilities.ts`, `agent-utils.ts`) import and depend on the `IAgent` interface rather than concrete agent classes
- No agent implementation files exist that lack the `api.public.contracts` export pattern or `IAgent` interface compliance
- All agent implementations that parse external configuration use standard validation patterns (`JSON.parse`, `parseTOML`) with error handling

<enforcement>
Clause Code MUST NOT skip or defer verification. TypeScript compiler type checking enforces `IAgent` interface compliance at build time. Code review must verify `api.public.contracts` export and `IAgent` implementation. Automated linting rules detect agent files missing `IAgent` interface imports. CI pipeline runs verification commands to count contract-compliant implementations. Build failures occur for non-compliant implementations. Code review blocks merge requests for agents lacking proper contract exports. Linting warnings are escalated to errors for missing `IAgent` imports. Quarterly architecture audits identify non-compliant legacy implementations.
</enforcement>
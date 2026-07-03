# Standardize Agent Public Contract Exports via IAgent Interface: Agent Implementations Extend

These rules are ALWAYS ACTIVE for all TypeScript modules in `src/agents/` that implement agent behavior, core utility modules that load or transform agent configurations, MCP integration modules that query agent capabilities, and agent utility functions that operate polymorphically on agent instances.

### Rules

- **R-AGENT-001** MUST: All agent implementation files in `src/agents/` export a class that implements `IAgent` or extends `AbstractAgent`/`AgentsMdAgent` base classes.
- **R-AGENT-002** MUST: Agent implementations export a public contract via the `api.public.contracts` facet for automated discovery by tools like `capabilities.ts` and `config-utils.ts`.
- **R-AGENT-003** MUST: Core utility modules (`config-utils.ts`, `capabilities.ts`, `agent-utils.ts`) import and depend on the `IAgent` interface rather than concrete agent classes.
- **R-AGENT-004** MUST: All agent implementations that parse external configuration use standard validation patterns (`JSON.parse`, `parseTOML`) wrapped in try-catch blocks with meaningful error messages including the configuration source path.
- **R-AGENT-005** MUST: When implementing `applyRulerConfig`, ensure the method is async and handles concurrent configuration updates safely to prevent race conditions.
- **R-AGENT-006** SHOULD: New agent implementations should start by importing `IAgent` from `'./IAgent'` and implementing all required methods before adding agent-specific logic.
- **R-AGENT-007** SHOULD: Use `AbstractAgent` or `AgentsMdAgent` base classes when agents share common functionality, but ensure the base class itself implements `IAgent`.
- **R-AGENT-008** MAY: Agent implementations MAY extend specialized base classes (`AbstractAgent`, `AgentsMdAgent`) to inherit common functionality while maintaining contract compliance.
- **R-AGENT-009** MAY: Experimental or deprecated agent implementations explicitly marked as non-standard in module documentation may receive temporary exceptions (max 2 quarters) before requiring full compliance or deprecation.

### Verify

```bash
# Count agent implementations that implement or extend IAgent/AbstractAgent/AgentsMdAgent
grep -r "implements.*IAgent\|extends.*AbstractAgent\|extends.*AgentsMdAgent" src/agents/*.ts | wc -l

# Count agent implementations with api.public.contracts export
grep -r "api.public.contracts=" src/agents/*.ts | wc -l

# Count IAgent imports across agent, core, and mcp modules
grep -r "import.*IAgent" src/agents/*.ts src/core/*.ts src/mcp/*.ts | wc -l

# Find agent implementation files that lack IAgent/AbstractAgent/AgentsMdAgent compliance
find src/agents -name "*Agent.ts" -exec grep -L "IAgent\|AbstractAgent\|AgentsMdAgent" {} \;
```

**Accept when:**
- All agent implementation files in `src/agents/` export a class that implements `IAgent` or extends `AbstractAgent`/`AgentsMdAgent`
- Core utility modules (`config-utils.ts`, `capabilities.ts`, `agent-utils.ts`) import and depend on the `IAgent` interface rather than concrete agent classes
- No agent implementation files exist that lack the `api.public.contracts` export pattern or `IAgent` interface compliance
- All agent implementations that parse external configuration use standard validation patterns (`JSON.parse`, `parseTOML`) with error handling

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler type checking enforces `IAgent` interface compliance at build time. Code review must verify `api.public.contracts` export and `IAgent` implementation. Automated linting rules detect agent files missing `IAgent` interface imports or implementations. CI pipeline runs grep-based verification commands. Build failures occur when agent implementations do not satisfy `IAgent` interface type constraints. Code review blocks merge requests for new agents lacking proper contract exports or interface compliance.
</enforcement>
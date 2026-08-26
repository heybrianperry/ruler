# Centralized IAgent Interface Module for Agent Contracts: Iagent Interface Module Located Shared Location

These rules are ALWAYS ACTIVE for all core infrastructure modules that coordinate agent behavior (configuration utilities, MCP capabilities, revert engine, agent selection) and any module that needs to type-check agent contracts without depending on concrete implementations.

### Rules

- **R-IAGENT-001** MUST: The IAgent interface module MUST be located in a shared location accessible via relative import from core infrastructure modules.
- **R-IAGENT-002** MUST: Core infrastructure modules that coordinate agent behavior MUST import the IAgent interface at the top of the module alongside other internal dependencies, using the established relative import pattern from the module's location to the interface definition.
- **R-IAGENT-003** MUST: Infrastructure modules MUST type function parameters and return values using the IAgent interface type to ensure compile-time verification of contract compliance; avoid using 'any' or overly generic types that bypass the contract.
- **R-IAGENT-004** MUST: When the IAgent interface needs to evolve, assess the impact on all importing modules by searching the codebase for interface imports before making changes; prefer additive changes (new optional methods or properties) over breaking changes to existing contracts.
- **R-IAGENT-005** SHOULD: Periodically review the interface for cohesion; consider splitting into multiple focused interfaces (e.g., IConfigurableAgent, IRevertableAgent) using interface composition if the contract grows too large.
- **R-IAGENT-006** SHOULD: Consider using path aliases or module resolution configuration to decouple import paths from physical file structure; document the expected module layout in architecture documentation.

### Verify

```bash
# Discover the project's module resolution configuration and verify that the IAgent interface module is accessible from core infrastructure locations using the documented import pattern.
grep -r "IAgent" --include="*.ts" --include="*.js" src/core/ | grep -E "(import|require).*IAgent"

# Search the codebase for all imports of the IAgent interface and verify that importing modules are in the core infrastructure scope (configuration, selection, capability detection, lifecycle management).
grep -r "from.*IAgent\|require.*IAgent" --include="*.ts" --include="*.js" | grep -E "(config|selection|capability|lifecycle)"

# Run the project's type checking tooling to verify that all IAgent interface usage in infrastructure modules satisfies the contract without type errors.
npm run type-check || yarn type-check || tsc --noEmit
```

**Accept when:**
- All core infrastructure modules that coordinate agent behavior successfully import and use the IAgent interface type without compilation or type-checking errors.
- No duplicate agent interface definitions exist in infrastructure modules; the centralized IAgent module is the single source of truth for agent contracts.
- Agent implementation modules implement the IAgent interface while infrastructure modules depend on it, establishing the correct dependency direction.
- Static type checking during continuous integration verifies that all IAgent interface usage satisfies the contract.

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checking failures that indicate incorrect IAgent interface usage block the build and must be resolved before merge. Code review MUST identify and reject changes that duplicate the interface or create direct dependencies on concrete agent classes.
</enforcement>
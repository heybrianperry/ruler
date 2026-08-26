# Centralized IAgent Interface Module for Agent Contracts: Core Infrastructure Modules Not Duplicate Agent

These rules are ALWAYS ACTIVE for all core infrastructure modules that coordinate agent behavior across configuration loading, capability detection, revert operations, and selection logic.

### Rules

- **R-AGENT-001** MUST NOT: Core infrastructure modules MUST NOT duplicate agent interface definitions locally or depend directly on concrete agent implementation classes.
- **R-AGENT-002** MUST: When creating new core infrastructure modules that coordinate agents, import the IAgent interface at the top of the module alongside other internal dependencies, using the established relative import pattern from the module's location to the interface definition.
- **R-AGENT-003** MUST: Infrastructure modules MUST type function parameters and return values using the IAgent interface type to ensure compile-time verification of contract compliance; avoid using 'any' or overly generic types that bypass the contract.
- **R-AGENT-004** SHOULD: When the IAgent interface needs to evolve, assess the impact on all importing modules by searching the codebase for interface imports before making changes; prefer additive changes (new optional methods or properties) over breaking changes to existing contracts.
- **R-AGENT-005** SHOULD: Periodically review the IAgent interface for cohesion; consider splitting into multiple focused interfaces (e.g., IConfigurableAgent, IRevertableAgent) using interface composition if the contract grows too large.

### Verify

```bash
# Discover the project's module resolution configuration and verify that the IAgent interface module is accessible from core infrastructure locations using the documented import pattern.
grep -r "IAgent" --include="*.ts" --include="*.js" . | grep -E "(config|selection|capability|lifecycle)" | head -20

# Search the codebase for all imports of the IAgent interface and verify that importing modules are in the core infrastructure scope.
grep -r "from.*IAgent" --include="*.ts" --include="*.js" . | grep -v "node_modules"

# Verify no duplicate agent interface definitions exist in infrastructure modules.
find . -path ./node_modules -prune -o -name "*.ts" -o -name "*.js" | xargs grep -l "interface IAgent" | grep -v agents/

# Run the project's type checking tooling to verify that all IAgent interface usage in infrastructure modules satisfies the contract without type errors.
npm run type-check || yarn type-check || tsc --noEmit
```

**Accept when:**
- All core infrastructure modules that coordinate agent behavior successfully import and use the IAgent interface type without compilation or type-checking errors.
- No duplicate agent interface definitions exist in infrastructure modules; the centralized IAgent module is the single source of truth for agent contracts.
- Agent implementation modules implement the IAgent interface while infrastructure modules depend on it, establishing the correct dependency direction.
- Type checking passes with no errors related to IAgent interface usage.
- Automated dependency analysis confirms the import graph maintains the correct dependency direction (infrastructure → interface ← implementations).

<enforcement>
Claude Code MUST NOT skip or defer verification. Static type checking during continuous integration MUST verify that all IAgent interface usage satisfies the contract. Code review MUST check that new infrastructure modules use the centralized interface rather than duplicating definitions or depending on concrete implementations. Type checking failures that indicate incorrect IAgent interface usage MUST block the build and be resolved before merge.
</enforcement>
# Centralized IAgent Interface Module for Agent Contracts: Modules That Perform Agent Configuration Selection

These rules are ALWAYS ACTIVE for all core infrastructure modules that coordinate agent behavior across configuration loading, capability detection, revert operations, and selection logic.

### Rules

- **R-IAGENT-001** SHOULD: Modules that perform agent configuration, selection, capability detection, or lifecycle management SHOULD depend on the IAgent interface rather than concrete implementations.
- **R-IAGENT-002** MUST: When creating new core infrastructure modules that coordinate agents, import the IAgent interface at the top of the module alongside other internal dependencies, using the established relative import pattern from the module's location to the interface definition.
- **R-IAGENT-003** MUST: Infrastructure modules MUST type function parameters and return values using the IAgent interface type to ensure compile-time verification of contract compliance; avoid using 'any' or overly generic types that bypass the contract.
- **R-IAGENT-004** SHOULD: When the IAgent interface needs to evolve, assess the impact on all importing modules by searching the codebase for interface imports before making changes; prefer additive changes (new optional methods or properties) over breaking changes to existing contracts.
- **R-IAGENT-005** MUST: Agent implementation modules MUST implement the IAgent interface while infrastructure modules depend on it, establishing the correct dependency direction (infrastructure → interface ← implementations).

### Verify

```bash
# Discover the project's module resolution configuration and verify that the IAgent interface module is accessible from core infrastructure locations using the documented import pattern.
grep -r "IAgent" --include="*.ts" --include="*.js" . | grep -E "(config|selection|capability|lifecycle)" | head -20

# Search the codebase for all imports of the IAgent interface and verify that importing modules are in the core infrastructure scope.
grep -r "from.*IAgent" --include="*.ts" --include="*.js" . | grep -v "node_modules"

# Run the project's type checking tooling to verify that all IAgent interface usage in infrastructure modules satisfies the contract without type errors.
npm run type-check || yarn type-check || tsc --noEmit

# Verify no duplicate agent interface definitions exist in infrastructure modules.
find . -name "*.ts" -o -name "*.js" | xargs grep -l "interface IAgent" | grep -v node_modules
```

**Accept when:**
- All core infrastructure modules that coordinate agent behavior successfully import and use the IAgent interface type without compilation or type-checking errors.
- No duplicate agent interface definitions exist in infrastructure modules; the centralized IAgent module is the single source of truth for agent contracts.
- Agent implementation modules implement the IAgent interface while infrastructure modules depend on it, establishing the correct dependency direction.
- Type checking passes without errors related to IAgent interface usage.
- All imports of IAgent originate from core infrastructure modules (configuration, selection, capability detection, lifecycle management) rather than from agent implementation modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checking failures that indicate incorrect IAgent interface usage block the build and must be resolved before merge. Code review MUST identify and reject changes that duplicate the interface or create direct dependencies on concrete agent classes.
</enforcement>
# Centralized IAgent Interface Module for Agent Contracts: Infrastructure Modules Import Iagent Interface Alongside

These rules are ALWAYS ACTIVE for all core infrastructure modules that coordinate agent behavior across configuration loading, capability detection, revert operations, and selection logic.

### Rules

- **R-IAGENT-001** MAY: Infrastructure modules MAY import the IAgent interface alongside other internal modules such as configuration loaders, utility functions, and constants as needed for their specific coordination responsibilities.
- **R-IAGENT-002** MUST: Infrastructure modules that coordinate agent behavior polymorphically MUST use the IAgent interface type for function parameters and return values to ensure compile-time verification of contract compliance.
- **R-IAGENT-003** MUST: The centralized IAgent module MUST be the single source of truth for agent contracts; duplicate agent interface definitions in infrastructure modules are prohibited.
- **R-IAGENT-004** MUST: Agent implementation modules MUST implement the IAgent interface; infrastructure modules MUST depend on it, establishing the correct dependency direction (infrastructure → interface ← implementations).
- **R-IAGENT-005** SHOULD: When the IAgent interface needs to evolve, prefer additive changes (new optional methods or properties) over breaking changes to existing contracts.
- **R-IAGENT-006** SHOULD: Infrastructure modules SHOULD avoid using 'any' or overly generic types that bypass the IAgent contract.
- **R-IAGENT-007** SHOULD: Periodically review the IAgent interface for cohesion; consider splitting into multiple focused interfaces using interface composition if the contract grows too large.

### Verify

```bash
# Discover the project's module resolution configuration and verify IAgent interface accessibility
grep -r "IAgent" --include="*.ts" --include="*.js" . | grep -E "(config-utils|mcp/capabilities|revert-engine|agent-selection)" | head -20

# Search for all imports of the IAgent interface
grep -r "from.*IAgent" --include="*.ts" --include="*.js" . | grep -v node_modules

# Verify no duplicate agent interface definitions exist in infrastructure modules
find . -path ./node_modules -prune -o -name "*.ts" -o -name "*.js" | xargs grep -l "interface IAgent" | grep -v agents/

# Run type checking to verify IAgent interface usage satisfies the contract
# (command varies by build tool; consult project's build configuration)
```

**Accept when:**
- All core infrastructure modules that coordinate agent behavior successfully import and use the IAgent interface type without compilation or type-checking errors.
- No duplicate agent interface definitions exist in infrastructure modules; the centralized IAgent module is the single source of truth for agent contracts.
- Agent implementation modules implement the IAgent interface while infrastructure modules depend on it, establishing the correct dependency direction.
- Static type checking during continuous integration verifies that all IAgent interface usage satisfies the contract.
- Code review confirms that new infrastructure modules use the centralized interface rather than duplicating definitions or depending on concrete implementations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checking failures that indicate incorrect IAgent interface usage block the build and must be resolved before merge. Code review MUST identify and reject changes that duplicate the interface or create direct dependencies on concrete agent classes.
</enforcement>
# Core Infrastructure Depends on Agent Abstraction Interface and Centralized Configuration Loader: Core Infrastructure Components Import Shared Constants

These rules are ALWAYS ACTIVE for all modules within the core infrastructure directory that coordinate, select, or orchestrate agent behavior, and for components that require configuration data to make decisions about agent usage or behavior.

### Rules

- **R-CORE-001** SHOULD: Core infrastructure components SHOULD import shared constants from the centralized constants module to maintain consistency across the core layer.
- **R-CORE-002** MUST: Core infrastructure components MUST depend on the IAgent interface abstraction rather than concrete agent implementation classes.
- **R-CORE-003** MUST: Core infrastructure components MUST use the centralized ConfigLoader module for all configuration access to ensure consistent configuration interpretation.
- **R-CORE-004** SHOULD: New core infrastructure components SHOULD replicate the established import patterns for IAgent and ConfigLoader observed in existing core modules.
- **R-CORE-005** MUST: New agent implementations MUST fully satisfy the IAgent interface contract before integrating with core infrastructure.
- **R-CORE-006** MUST: When modifying the IAgent interface, all agent implementations and core infrastructure usage sites MUST be audited and updated in a coordinated manner.

### Verify

```bash
# Discover the project's module resolution configuration
find . -name "tsconfig.json" -o -name "jsconfig.json" -o -name "package.json" | head -5

# Verify relative imports from core infrastructure resolve correctly
grep -r "from.*IAgent" ./core --include="*.ts" --include="*.js" | head -10
grep -r "from.*ConfigLoader" ./core --include="*.ts" --include="*.js" | head -10

# Run static analysis or type checking tooling
# (Exact command depends on project build tool — check package.json or build config)
npm run type-check 2>&1 || yarn type-check 2>&1 || tsc --noEmit 2>&1

# Verify no direct imports of concrete agent classes in core infrastructure
grep -r "from.*Agent[^I]" ./core --include="*.ts" --include="*.js" | grep -v "IAgent" | grep -v "AgentAbstraction"

# Run core infrastructure test suite
npm test -- --testPathPattern="core" 2>&1 || yarn test --testPathPattern="core" 2>&1
```

**Accept when:**
- Static analysis confirms all core infrastructure modules import IAgent interface rather than concrete agent classes
- Type checking verifies that all uses of IAgent and ConfigLoader in core infrastructure satisfy their respective interface contracts
- No direct imports of concrete agent implementation classes are found in core infrastructure modules
- Test suite passes for core infrastructure components, confirming that agent abstraction dependencies integrate correctly
- Relative imports from core infrastructure to agent abstractions resolve without errors

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code changes affecting core infrastructure components. Type checking and static analysis violations MUST block merge until corrected.
</enforcement>
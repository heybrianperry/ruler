# Core Infrastructure Depends on Agent Abstraction Interface and Centralized Configuration Loader: Core Infrastructure Components Depend Iagent Interface

These rules are ALWAYS ACTIVE for all core infrastructure components responsible for agent orchestration, agent selection logic, and coordination of multiple agent implementations.

### Rules

- **R-CORE-001** MUST: Core infrastructure components MUST depend on the IAgent interface abstraction rather than concrete agent implementation classes.
- **R-CORE-002** MUST: All modules within the core infrastructure directory that coordinate, select, or orchestrate agent behavior MUST use the IAgent interface for agent interactions.
- **R-CORE-003** MUST: Components that require configuration data to make decisions about agent usage or behavior MUST use the centralized ConfigLoader module.
- **R-CORE-004** SHOULD: When adding new core infrastructure components, examine existing core modules to identify and replicate the established import patterns for IAgent and ConfigLoader.
- **R-CORE-005** SHOULD: When implementing new agent types, ensure they fully satisfy the IAgent interface contract before integrating with core infrastructure.
- **R-CORE-006** SHOULD: When modifying the IAgent interface, audit all agent implementations and core infrastructure usage sites to ensure coordinated updates across the dependency boundary.

### Verify

```bash
# Discover the project's module resolution configuration and verify relative imports
find . -path ./node_modules -prune -o -type f -name '*.ts' -o -name '*.js' | xargs grep -l 'core.*infrastructure' | head -20

# Locate and execute static analysis or type checking tooling
# (Exact command depends on project build tool — check package.json or build config)
npm run type-check 2>&1 || yarn type-check 2>&1 || echo "Type checker not found"

# Verify core infrastructure imports reference IAgent and ConfigLoader, not concrete implementations
grep -r "from.*agent" --include="*.ts" --include="*.js" | grep -E "(revert-engine|agent-selection)" | grep -v "IAgent" | grep -v "ConfigLoader"

# Run test suite for core infrastructure components
npm test -- --testPathPattern="(revert-engine|agent-selection|core)" 2>&1 || yarn test --testPathPattern="(revert-engine|agent-selection|core)" 2>&1
```

**Accept when:**
- Static analysis confirms all core infrastructure modules import IAgent interface rather than concrete agent classes
- Type checking verifies that all uses of IAgent and ConfigLoader in core infrastructure satisfy their respective interface contracts
- Test suite passes for core infrastructure components, confirming that agent abstraction dependencies integrate correctly
- No direct imports of concrete agent implementation classes are found in core infrastructure modules

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for core infrastructure components. Violations block merge until corrected.
</enforcement>
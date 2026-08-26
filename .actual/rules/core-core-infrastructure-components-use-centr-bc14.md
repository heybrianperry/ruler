# Core Infrastructure Depends on Agent Abstraction Interface and Centralized Configuration Loader: Core Infrastructure Components Use Centralized Configloader

These rules are ALWAYS ACTIVE for all core infrastructure components responsible for agent orchestration, agent selection logic, and coordination of multiple agent implementations.

### Rules

- **R-CORE-001** MUST: Core infrastructure components MUST use the centralized ConfigLoader module for all configuration access rather than implementing independent configuration loading logic.

### Verify

```bash
# Discover the project's module resolution configuration and verify that relative imports from core infrastructure to agent abstractions resolve correctly
find . -name "*.ts" -o -name "*.js" | xargs grep -l "from.*IAgent\|from.*ConfigLoader" | head -20

# Locate and execute the project's static analysis or type checking tooling to verify that all core infrastructure imports of IAgent and ConfigLoader satisfy type contracts
npm run type-check 2>&1 | grep -E "(IAgent|ConfigLoader|core/)"

# Identify the project's test suite and run tests covering core infrastructure components to verify that agent abstraction dependencies function correctly
npm test -- --testPathPattern="(core|infrastructure)" 2>&1 | tail -20
```

**Accept when:**
- Static analysis confirms all core infrastructure modules import IAgent interface rather than concrete agent classes
- Type checking verifies that all uses of IAgent and ConfigLoader in core infrastructure satisfy their respective interface contracts
- Test suite passes for core infrastructure components, confirming that agent abstraction dependencies integrate correctly

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification commands must execute successfully before accepting changes to core infrastructure components.
</enforcement>
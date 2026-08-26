# Core Infrastructure Depends on Agent Abstraction Interface and Centralized Configuration Loader: Dependencies Core Infrastructure Agent Abstractions Use

These rules are ALWAYS ACTIVE for all modules within the core infrastructure directory that coordinate, select, or orchestrate agent behavior, and for components that require configuration data to make decisions about agent usage or behavior.

### Rules

- **R-CORE-DEP-001** MUST: Dependencies from core infrastructure to agent abstractions MUST use relative imports that traverse upward to the agents module boundary.

### Verify

```bash
# Discover the project's module resolution configuration and verify that relative imports from core infrastructure to agent abstractions resolve correctly
find . -path ./node_modules -prune -o -type f \( -name '*.ts' -o -name '*.js' \) -print | xargs grep -l 'from.*\.\./.*agents' | head -20

# Locate and execute the project's static analysis or type checking tooling to verify that all core infrastructure imports of IAgent and ConfigLoader satisfy type contracts
# (Requires project-specific build tool discovery from dependency manifest)

# Identify the project's test suite and run tests covering core infrastructure components to verify that agent abstraction dependencies function correctly
# (Requires project-specific test runner discovery)
```

**Accept when:**
- Static analysis confirms all core infrastructure modules import IAgent interface rather than concrete agent classes
- Type checking verifies that all uses of IAgent and ConfigLoader in core infrastructure satisfy their respective interface contracts
- Test suite passes for core infrastructure components, confirming that agent abstraction dependencies integrate correctly

<enforcement>
Claude Code MUST NOT skip or defer verification. All core infrastructure imports must be audited against this rule before acceptance.
</enforcement>
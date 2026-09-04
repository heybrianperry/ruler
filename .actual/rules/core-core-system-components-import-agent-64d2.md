# Core Agent Modules Centralize Interface Contracts and Configuration Management: Core System Components Import Agent Interface

These rules are ALWAYS ACTIVE for all core system components that interact with agent implementations, require configuration loading, or orchestrate agent behavior across the detection pipeline.

### Rules

- **R-CORE-001** MUST: Core system components MUST import agent interface contracts from the centralized IAgent module rather than defining local agent interfaces.
- **R-CORE-002** MUST: All modules within the core system directory that interact with agent implementations MUST use centralized IAgent imports.
- **R-CORE-003** MUST: Components that require configuration loading MUST use the centralized ConfigLoader rather than implementing duplicate configuration parsing logic.
- **R-CORE-004** MUST: Engine implementations that orchestrate agent behavior (selection, revert, apply) MUST import shared agent modules from the centralized location.
- **R-CORE-005** SHOULD: New core components SHOULD identify required shared modules by examining existing core components in the same directory before implementation.
- **R-CORE-006** SHOULD: Any new agent capabilities SHOULD be added to the IAgent interface rather than creating parallel interfaces.
- **R-CORE-007** SHOULD: If a core component requires configuration, verify that ConfigLoader supports the required configuration schema before implementing the component.

### Verify

```bash
# Discover the project's module resolution configuration and verify relative imports resolve correctly
grep -r "paths\|baseUrl" tsconfig.json 2>/dev/null || echo "Check TypeScript module resolution config"

# Locate and execute static analysis tools to detect circular dependencies
npm run lint 2>/dev/null || yarn lint 2>/dev/null || echo "Run project linting tool"

# Verify TypeScript type checking passes for all IAgent and ConfigLoader imports
npm run type-check 2>/dev/null || yarn type-check 2>/dev/null || npx tsc --noEmit 2>/dev/null || echo "Run TypeScript compiler"

# Search for any local agent interface definitions that should be centralized
grep -r "interface IAgent" --include="*.ts" --exclude-dir=node_modules | grep -v "core/agent" || echo "No duplicate IAgent interfaces found"

# Verify no circular dependencies exist in core modules
grep -r "import.*from.*\.\." core/ --include="*.ts" | head -20 || echo "Check relative import patterns"
```

**Accept when:**
- All core components successfully import IAgent, ConfigLoader, and agent-utils without compilation errors
- Static analysis confirms no circular dependencies exist between core modules and their dependencies
- Type checking passes for all usages of imported interfaces and utilities, confirming contract compliance
- No duplicate IAgent interface definitions exist outside the centralized core/agent module
- All relative imports in core modules resolve to intended centralized modules

<enforcement>
Claude Code MUST NOT skip or defer verification. Build failures and type-checking errors MUST block implementation. Code review MUST verify centralized module usage before approval.
</enforcement>
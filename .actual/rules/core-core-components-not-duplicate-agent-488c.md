# Core Agent Modules Centralize Interface Contracts and Configuration Management: Core Components Not Duplicate Agent Interface

These rules are ALWAYS ACTIVE for all core system components that interact with agent implementations, require configuration loading, or orchestrate agent behavior across the core system directory.

### Rules

- **R-CORE-001** MUST NOT: Core components MUST NOT duplicate agent interface definitions or configuration loading logic that already exists in centralized modules.
- **R-CORE-002** MUST: Maintain the IAgent interface as the single source of truth for agent contracts—any new agent capabilities MUST be added to this interface rather than creating parallel interfaces.
- **R-CORE-003** MUST: When a core component requires configuration, verify that ConfigLoader supports the required configuration schema before implementing the component.
- **R-CORE-004** SHOULD: Identify which shared modules are required by examining existing core components in the same directory when implementing new core components.
- **R-CORE-005** SHOULD: Use relative path imports to make dependency relationships explicit and enable static analysis tools to verify module boundaries.

### Verify

```bash
# Discover the project's module resolution configuration and verify relative imports resolve correctly
grep -r "paths\|baseUrl" tsconfig.json || echo "No path aliases configured"

# Locate and execute static analysis or linting tools to detect circular dependencies
npm run lint 2>&1 | grep -i "circular\|import" || echo "No circular dependency issues detected"

# Identify the project's type checking configuration and verify interface contracts
npm run type-check 2>&1 | grep -E "IAgent|ConfigLoader" || echo "Type checking passed for core modules"

# Verify all core components successfully import IAgent and ConfigLoader without errors
grep -r "import.*IAgent\|import.*ConfigLoader" src/core/ | wc -l
```

**Accept when:**
- All core components successfully import IAgent, ConfigLoader, and agent-utils without compilation errors
- Static analysis confirms no circular dependencies exist between core modules and their dependencies
- Type checking passes for all usages of imported interfaces and utilities, confirming contract compliance
- No duplicate agent interface definitions or configuration loading logic exist across core components

<enforcement>
Claude Code MUST NOT skip or defer verification. Build failures and type-checking errors MUST block deployment. Code review MUST verify that new core components use centralized modules rather than duplicating functionality.
</enforcement>
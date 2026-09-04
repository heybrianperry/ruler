# Core Agent Modules Centralize Interface Contracts and Configuration Management: Core Modules Use Relative Path Imports

These rules are ALWAYS ACTIVE for all core system modules that interact with agent implementations, require configuration loading, or orchestrate agent behavior across different operational contexts.

### Rules

- **R-CORE-001** SHOULD: Core modules SHOULD use relative path imports to establish explicit dependency relationships between components.
- **R-CORE-002** MUST: Maintain the IAgent interface as the single source of truth for agent contracts—any new agent capabilities MUST be added to this interface rather than creating parallel interfaces.
- **R-CORE-003** MUST: All core components MUST successfully import IAgent, ConfigLoader, and agent-utils without compilation errors.
- **R-CORE-004** MUST: When implementing new core components, identify which shared modules are required by examining existing core components in the same directory.
- **R-CORE-005** MUST: If a core component requires configuration, verify that ConfigLoader supports the required configuration schema before implementing the component.
- **R-CORE-006** MUST: Static analysis tools MUST detect and prevent circular dependencies between core modules and their dependencies during continuous integration.
- **R-CORE-007** MUST: Code review MUST verify that new core components use centralized modules rather than duplicating functionality.

### Verify

```bash
# Discover the project's module resolution configuration and verify relative imports resolve correctly
grep -r "paths\|baseUrl" tsconfig.json || echo "Check TypeScript module resolution config"

# Locate and execute static analysis tools to detect circular dependencies
npm run lint 2>&1 | grep -i "circular\|import" || echo "Run project linting"

# Verify TypeScript type checking passes for all imports
npm run type-check 2>&1 | grep -E "error|Error" || echo "Type checking passed"

# Confirm all core components import from centralized modules
grep -r "from.*['\"]\.\..*IAgent\|ConfigLoader" src/core/ || echo "Verify centralized imports in core modules"

# Check for circular dependency detection in build configuration
grep -r "circular" .eslintrc* package.json || echo "Verify circular dependency linting rules"
```

**Accept when:**
- All core components successfully import IAgent, ConfigLoader, and agent-utils without compilation errors
- Static analysis confirms no circular dependencies exist between core modules and their dependencies
- Type checking passes for all usages of imported interfaces and utilities, confirming contract compliance
- New core components use centralized modules rather than duplicating functionality
- Module resolution configuration correctly maps relative imports to intended core modules

<enforcement>
Claude Code MUST NOT skip or defer verification. Build failures and type-checking errors MUST block deployment. Code review MUST enforce adherence to centralized module patterns before merge.
</enforcement>
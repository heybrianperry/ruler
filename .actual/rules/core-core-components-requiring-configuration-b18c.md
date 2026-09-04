# Core Agent Modules Centralize Interface Contracts and Configuration Management: Core Components Requiring Configuration Access Use

These rules are ALWAYS ACTIVE for all core system components that interact with agent implementations, require configuration loading, or orchestrate agent behavior across the core system directory.

### Rules

- **R-CORE-001** MUST: Core components requiring configuration access MUST use the ConfigLoader module rather than implementing independent configuration loading logic.
- **R-CORE-002** MUST: All core components MUST import and use the IAgent interface as the single source of truth for agent contracts rather than creating parallel interfaces.
- **R-CORE-003** MUST: Core components MUST NOT duplicate agent interface definitions or configuration parsing logic already present in centralized modules.
- **R-CORE-004** SHOULD: New core components SHOULD identify required shared modules by examining existing core components in the same directory before implementation.
- **R-CORE-005** SHOULD: Configuration schema requirements SHOULD be verified against ConfigLoader capabilities before implementing new core components.

### Verify

```bash
# Discover the project's module resolution configuration and verify relative imports resolve correctly
grep -r "baseUrl\|paths" tsconfig.json || echo "Check module resolution in build config"

# Locate and execute static analysis tools to detect circular dependencies
npm run lint 2>&1 | grep -i "circular\|import" || echo "No circular dependency warnings"

# Verify TypeScript type checking passes for all IAgent and ConfigLoader imports
npm run type-check 2>&1 | grep -E "(IAgent|ConfigLoader)" || echo "Type checking passed"

# Confirm all core components successfully import required modules without errors
find . -path ./node_modules -prune -o -name "*.ts" -type f -exec grep -l "import.*IAgent\|import.*ConfigLoader" {} \; | head -20
```

**Accept when:**
- All core components successfully import IAgent, ConfigLoader, and agent-utils without compilation errors
- Static analysis confirms no circular dependencies exist between core modules and their dependencies
- Type checking passes for all usages of imported interfaces and utilities, confirming contract compliance
- No core component duplicates configuration loading logic or agent interface definitions
- All new core components reference centralized modules rather than implementing parallel functionality

<enforcement>
Claude Code MUST NOT skip or defer verification. Build failures, type-check failures, or linting errors indicating circular dependencies or interface contract violations MUST block implementation. Code review MUST verify centralized module usage before accepting new core components.
</enforcement>
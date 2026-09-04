# Core Agent Modules Centralize Interface Contracts and Configuration Management: Shared Agent Utilities Imported Centralized Utils

These rules are ALWAYS ACTIVE for all modules within the core system directory that interact with agent implementations, components that require configuration loading or access to system constants, and engine implementations that orchestrate agent behavior.

### Rules

- **R-CORE-001** MUST: Shared agent utilities MUST be imported from the centralized agent-utils module to ensure consistent behavior across components.
- **R-CORE-002** MUST: Maintain the IAgent interface as the single source of truth for agent contracts—any new agent capabilities should be added to this interface rather than creating parallel interfaces.
- **R-CORE-003** MUST: All core components must successfully import IAgent, ConfigLoader, and agent-utils without compilation errors.
- **R-CORE-004** SHOULD: When implementing new core components, identify which shared modules are required by examining existing core components in the same directory.
- **R-CORE-005** SHOULD: If a core component requires configuration, verify that ConfigLoader supports the required configuration schema before implementing the component.
- **R-CORE-006** MAY: Use path aliases or module resolution configuration to decouple import paths from physical directory structure during refactoring.

### Verify

```bash
# Discover the project's module resolution configuration and verify relative imports resolve correctly
grep -r "paths\|baseUrl" tsconfig.json || echo "Check module resolution in build config"

# Locate and execute static analysis tools to detect circular dependencies
npm run lint 2>/dev/null || yarn lint 2>/dev/null || echo "Run project linting tool"

# Verify TypeScript type checking passes for all imports
npm run type-check 2>/dev/null || yarn type-check 2>/dev/null || npx tsc --noEmit

# Confirm no circular dependencies between core modules
grep -r "import.*from.*core" src/core/ | sort | uniq
```

**Accept when:**
- All core components successfully import IAgent, ConfigLoader, and agent-utils without compilation errors
- Static analysis confirms no circular dependencies exist between core modules and their dependencies
- Type checking passes for all usages of imported interfaces and utilities, confirming contract compliance
- No parallel agent interfaces exist outside the centralized IAgent definition

<enforcement>
Claude Code MUST NOT skip or defer verification. Build failures and type-checking errors MUST block deployment. Code review MUST verify that new core components use centralized modules rather than duplicating functionality.
</enforcement>
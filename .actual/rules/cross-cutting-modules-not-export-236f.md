# Standardize Agent and Utility Module Exports as Public API Contracts: Modules Not Export

These rules are ALWAYS ACTIVE for all agent implementation files in `src/agents/` and core utility modules in `src/core/` that expose public interfaces, including interface definitions, base classes, type definitions, and configuration/processing modules.

### Rules

- **R-EXPORT-001** MUST_NOT: Modules MUST NOT export internal implementation details, helper functions, or types that are not intended for external consumption.
- **R-EXPORT-002** MUST: All agent implementation files MUST export a named class that implements IAgent or extends AbstractAgent/AgentsMdAgent.
- **R-EXPORT-003** MUST: Core utility modules (path-utils, SubagentsUtils, SubagentsProcessor, config-utils, agent-selection) MUST export documented public functions with clear names.
- **R-EXPORT-004** MUST: Import statements in agent files MUST follow established relative path conventions ('./IAgent', '../core/ModuleName').
- **R-EXPORT-005** SHOULD: Document public contracts with JSDoc comments including @public tag, parameter descriptions, and return types.
- **R-EXPORT-006** SHOULD: Use TypeScript export keyword to explicitly mark classes, functions, and types as public contracts.

### Verify

```bash
# Verify agent implementations export named classes
grep -r 'export class.*Agent' src/agents/ | wc -l

# Verify core utility exports are present
grep -r 'export.*function' src/core/ | grep -E '(parseFrontmatter|validateFrontmatter|loadSubagentFile|isPathInsideOrEqual|discoverSubagents|resolveSelectedAgents)'

# Verify agents import IAgent interface
grep -r "from '\./IAgent'" src/agents/ | wc -l

# Verify agents import from core utilities
grep -r "from '\.\.\/core/" src/agents/ | wc -l
```

**Accept when:**
- All agent implementation files export a named class that implements IAgent or extends AbstractAgent/AgentsMdAgent
- Core utility modules export documented public functions with clear names
- Import statements in agent files follow the established relative path conventions ('./IAgent', '../core/ModuleName')
- No internal implementation details or undocumented helper functions are exported from modules
- TypeScript compiler successfully validates interface implementation and type correctness

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. TypeScript compilation errors block merge if interface contracts are violated. Code review feedback MUST request changes to align with export and import conventions. CI pipeline MUST fail if verification commands do not pass expected thresholds.
</enforcement>
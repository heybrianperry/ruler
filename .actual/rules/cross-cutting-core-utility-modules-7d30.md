# Standardize Agent and Utility Module Exports as Public API Contracts: Core Utility Modules

These rules are ALWAYS ACTIVE for all agent implementation files in `src/agents/` and all core utility modules in `src/core/` that provide shared functionality, including interface definitions, base classes, type definitions, and configuration/processing modules.

### Rules

- **R-CORE-001** MUST: Core utility modules MUST explicitly export all public functions and types intended for consumption by agents or other modules (e.g., parseFrontmatter, validateFrontmatter, loadSubagentFile, isPathInsideOrEqual, discoverSubagents, resolveSelectedAgents).
- **R-CORE-002** MUST: All agent implementation files MUST export a named class that implements IAgent or extends AbstractAgent/AgentsMdAgent.
- **R-CORE-003** MUST: Agent implementations MUST import shared interfaces from './IAgent' and base classes from './AbstractAgent' or './AgentsMdAgent' using relative path conventions.
- **R-CORE-004** MUST: Core utility modules MUST use specific named exports rather than default exports for public functions and types.
- **R-CORE-005** SHOULD: Public contracts SHOULD be documented with JSDoc comments including @public tag, parameter descriptions, and return types.
- **R-CORE-006** SHOULD: New agent types SHOULD extend AbstractAgent or AgentsMdAgent base classes and implement required IAgent interface methods to maintain consistency.

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
- Core utility modules (path-utils, SubagentsUtils, SubagentsProcessor, config-utils, agent-selection) export documented public functions with clear names
- Import statements in agent files follow the established relative path conventions ('./IAgent', '../core/ModuleName')
- No internal implementation details or undocumented helper functions are exported from modules
- TypeScript compilation succeeds with no interface implementation errors

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. TypeScript compiler checks and code review verification are mandatory before accepting changes to agent or core utility module exports.
</enforcement>
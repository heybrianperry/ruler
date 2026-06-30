# Standardize Agent and Utility Module Exports as Public API Contracts: Agent Implementations Extend

These rules are ALWAYS ACTIVE for all agent implementations in `src/agents/` and core utility modules in `src/core/` that expose public interfaces, including interface definitions (IAgent) and base classes (AbstractAgent, AgentsMdAgent).

### Rules

- **R-AGENT-001** MUST: Agent implementations extend base agent classes (AbstractAgent, AgentsMdAgent) and implement the IAgent interface with explicit `export class` declarations.
- **R-AGENT-002** MUST: Core utility modules export specific named functions and types as public contracts rather than default exports (e.g., `export function discoverSubagents()`, `export function resolveSelectedAgents()`).
- **R-AGENT-003** MUST: Agent implementation files import shared interfaces and base classes using relative paths (`./IAgent`, `./AbstractAgent`, `./AgentsMdAgent`) and core utilities using relative paths (`../core/ModuleName`).
- **R-AGENT-004** MAY: Agent implementations MAY extend the base agent classes with additional public methods specific to their integration requirements.
- **R-AGENT-005** SHOULD: Document all public contracts with JSDoc comments including `@public` tag, parameter descriptions, and return types to clarify intended usage.
- **R-AGENT-006** SHOULD: Use `@experimental` or `@internal` JSDoc tags to explicitly mark APIs that deviate from standard export patterns or are not intended for general consumption.

### Verify

```bash
# Verify all agent implementations export a named class
grep -r 'export class.*Agent' src/agents/ | wc -l

# Verify core utility exports are present
grep -r 'export.*function' src/core/ | grep -E '(parseFrontmatter|validateFrontmatter|loadSubagentFile|isPathInsideOrEqual|discoverSubagents|resolveSelectedAgents)'

# Verify agents import IAgent interface
grep -r "from './IAgent'" src/agents/ | wc -l

# Verify agents import from core utilities
grep -r "from '../core/" src/agents/ | wc -l

# Verify no circular dependencies in import patterns
grep -r "from '\./" src/core/ | grep -v test | wc -l
```

**Accept when:**
- All agent implementation files export a named class that implements IAgent or extends AbstractAgent/AgentsMdAgent
- Core utility modules (path-utils, SubagentsUtils, SubagentsProcessor, config-utils, agent-selection) export documented public functions with clear names
- Import statements in agent files follow the established relative path conventions (`./IAgent`, `../core/ModuleName`)
- No internal implementation details or undocumented helper functions are exported from modules
- Public API contracts are documented with JSDoc comments including parameter and return type information
- TypeScript compilation succeeds without interface implementation errors

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All agent implementations and core utility modules MUST comply with the export and import patterns defined above. TypeScript compiler checks and code review verification are mandatory before accepting changes to public API contracts.
</enforcement>
# Standardize Agent and Utility Module Exports as Public API Contracts: Modules That Coordinate

These rules are ALWAYS ACTIVE for all agent implementation files in `src/agents/`, all core utility modules in `src/core/` that provide shared functionality, interface definitions (IAgent), base classes (AbstractAgent, AgentsMdAgent), type definitions shared across modules, and configuration/processing modules that coordinate agent behavior.

### Rules

- **R-COORD-001** SHOULD: Modules that coordinate agent lifecycle (discovery, selection, propagation) SHOULD export functions with clear, action-oriented names that describe their purpose (e.g., `discoverSubagents`, `resolveSelectedAgents`, `getSelectedSubagentTargets`).

### Verify

```bash
# Verify agent implementations export named classes implementing IAgent or extending base classes
grep -r 'export class.*Agent' src/agents/ | wc -l

# Verify core utility exports include expected public functions
grep -r 'export.*function' src/core/ | grep -E '(parseFrontmatter|validateFrontmatter|loadSubagentFile|isPathInsideOrEqual|discoverSubagents|resolveSelectedAgents)'

# Verify agents import IAgent interface using relative paths
grep -r "from '\./IAgent'" src/agents/ | wc -l

# Verify agents import from core utilities using relative paths
grep -r "from '\.\.\/core/" src/agents/ | wc -l
```

**Accept when:**
- All agent implementation files export a named class that implements IAgent or extends AbstractAgent/AgentsMdAgent
- Core utility modules (path-utils, SubagentsUtils, SubagentsProcessor, config-utils, agent-selection) export documented public functions with clear, action-oriented names
- Import statements in agent files follow established relative path conventions (`./IAgent`, `../core/ModuleName`)
- No internal implementation details or undocumented helper functions are exported from modules
- Public contracts are documented with JSDoc comments including `@public` tag, parameter descriptions, and return types

<enforcement>
Clause Code MUST NOT skip or defer verification. TypeScript compiler checks for interface implementation and type correctness are mandatory. Code review must verify public export patterns and import conventions. ESLint rules must enforce relative import path patterns. CI pipeline must execute grep-based verification commands and fail if expected thresholds are not met.
</enforcement>
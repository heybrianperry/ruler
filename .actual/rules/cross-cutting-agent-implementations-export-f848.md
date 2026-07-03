# Standardize Agent and Utility Module Exports as Public API Contracts: Agent Implementations Export

These rules are ALWAYS ACTIVE for all agent implementations in `src/agents/` and core utility modules in `src/core/` that expose public interfaces.

### Rules

- **R-AGENT-001** MUST: All agent implementations MUST export a named class that implements the IAgent interface or extends AbstractAgent/AgentsMdAgent base classes.

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

<enforcement>
Clause Code MUST NOT skip or defer verification. TypeScript compiler checks for interface implementation and type correctness. Code review checklist includes verification of public export patterns and import conventions. ESLint rules enforce relative import path patterns and prevent circular dependencies. Automated grep-based verification in CI pipeline checks for expected export patterns.
</enforcement>
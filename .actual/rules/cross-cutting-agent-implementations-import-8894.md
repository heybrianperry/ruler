# Standardize Agent and Utility Module Exports as Public API Contracts: Agent Implementations Import

These rules are ALWAYS ACTIVE for all agent implementation files in `src/agents/` directory, all core utility modules in `src/core/` directory that provide shared functionality, interface definitions (IAgent) and base classes (AbstractAgent, AgentsMdAgent), type definitions and data structures shared across modules, and configuration and processing modules that coordinate agent behavior.

### Rules

- **R-AGENT-EXPORT-001** MUST: Agent implementations MUST import from core modules using the established relative path conventions ('./IAgent' for same directory, '../core/' for core utilities).

### Verify

```bash
# Verify agent implementations export named classes
grep -r 'export class.*Agent' src/agents/ | wc -l

# Verify core utility exports are present
grep -r 'export.*function' src/core/ | grep -E '(parseFrontmatter|validateFrontmatter|loadSubagentFile|isPathInsideOrEqual|discoverSubagents|resolveSelectedAgents)'

# Verify agents import IAgent interface using correct relative path
grep -r "from './IAgent'" src/agents/ | wc -l

# Verify agents import from core utilities using correct relative path
grep -r "from '../core/" src/agents/ | wc -l
```

**Accept when:**
- All agent implementation files export a named class that implements IAgent or extends AbstractAgent/AgentsMdAgent
- Core utility modules (path-utils, SubagentsUtils, SubagentsProcessor, config-utils, agent-selection) export documented public functions with clear names
- Import statements in agent files follow the established relative path conventions ('./IAgent', '../core/ModuleName')
- No internal implementation details or undocumented helper functions are exported from modules
- TypeScript compiler confirms all interface implementations and type correctness

<enforcement>
Claude Code MUST NOT skip or defer verification. All verification commands MUST pass before accepting changes to agent implementations or core utility modules. TypeScript compilation errors block merge if interface contracts are violated. Code review feedback MUST request changes to align with export and import conventions. CI pipeline MUST fail if verification commands do not pass expected thresholds.
</enforcement>
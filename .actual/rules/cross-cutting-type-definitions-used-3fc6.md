# Standardize Agent and Utility Module Exports as Public API Contracts: Type Definitions Used

These rules are ALWAYS ACTIVE for all agent implementation files in `src/agents/` and core utility modules in `src/core/` that expose public interfaces, including interface definitions (IAgent) and base classes (AbstractAgent, AgentsMdAgent).

### Rules

- **R-EXPORTS-001** SHOULD: Type definitions used across multiple modules (ParsedFrontmatter, CopilotToolMapping) SHOULD be exported from a single authoritative module to prevent duplication.

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
- Type definitions shared across modules are centralized in a single authoritative location

<enforcement>
TypeScript compiler checks for interface implementation and type correctness are mandatory. Code review must verify public export patterns and import conventions. ESLint rules must enforce relative import path patterns. Automated grep-based verification in CI pipeline must check for expected export patterns. TypeScript compilation errors block merge if interface contracts are violated. CI pipeline fails if verification commands do not pass expected thresholds.
</enforcement>
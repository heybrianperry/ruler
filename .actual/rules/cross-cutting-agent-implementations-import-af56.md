# Standardize Core Library Imports for Agent and Utility Modules: Agent Implementations Import

These rules are ALWAYS ACTIVE for all agent implementations in `src/agents/` and core utility modules in `src/core/` that handle file system operations, configuration loading, and agent discovery.

### Rules

- **R-AGENT-IMP-001** MUST: Agent implementations MUST import the 'path' module from Node.js core for all file path operations.
- **R-AGENT-IMP-002** MUST: Agent implementations MUST export their class as a public contract for external consumers.
- **R-AGENT-IMP-003** MUST: All JSON.parse calls in agent implementations and core utilities MUST be wrapped in try-catch blocks or equivalent error handling logic.
- **R-AGENT-IMP-004** SHOULD: New agent implementations should extend AbstractAgent or AgentsMdAgent base classes to inherit standard import patterns.
- **R-AGENT-IMP-005** SHOULD: Use 'fs/promises' instead of callback-based 'fs' for new code to enable async/await patterns.
- **R-AGENT-IMP-006** SHOULD: Centralize configuration parsing logic in core utility modules rather than duplicating JSON.parse calls in individual agents.

### Verify

```bash
# Verify all agent implementations import 'path' from Node.js core
grep -r "import.*from 'path'" src/agents/ src/core/ | wc -l

# Verify all agent implementations import 'fs' or 'fs/promises'
grep -r "import.*from 'fs" src/agents/ src/core/ | wc -l

# Verify all agent implementations export their class
grep -r "export.*class.*Agent" src/agents/ | wc -l

# Verify JSON.parse calls are protected with error handling
grep -r "JSON.parse" src/agents/ src/core/ | grep -v "try" | wc -l
```

**Accept when:**
- All agent implementations in `src/agents/` import 'path' from Node.js core
- All agent implementations export their class as a public contract
- All JSON.parse calls are wrapped in try-catch blocks or error handling logic
- Core utility modules export named functions for shared functionality (parseFrontmatter, loadSubagentFile, discoverSubagents)
- No unprotected JSON.parse calls exist in agent or core utility modules

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for agent implementations and core utility modules. Violations must be resolved before merge to main branch. Exceptions require architecture review approval from two senior engineers.
</enforcement>
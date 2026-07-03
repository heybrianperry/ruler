# Standardize Core Library Imports for Agent and Utility Modules: Agent Implementations Import

These rules are ALWAYS ACTIVE for all agent implementations and core utility modules within the codebase, including all files in `src/agents/` and `src/core/` directories that handle agent instantiation, configuration loading, and file system operations.

### Rules

- **R-AGENT-001** MUST: Agent implementations MUST import and implement either './IAgent' interface or extend './AbstractAgent' or './AgentsMdAgent' base classes.
- **R-AGENT-002** MUST: All agent implementations in src/agents/ MUST import 'path' from Node.js core.
- **R-AGENT-003** MUST: All agent implementations in src/agents/ MUST import 'fs' or 'fs/promises' from Node.js core.
- **R-AGENT-004** MUST: All agent implementations MUST export their class as a public contract.
- **R-AGENT-005** MUST: All JSON.parse calls in agent and utility modules MUST be wrapped in try-catch blocks or equivalent error handling logic.
- **R-AGENT-006** MUST: Core utility modules MUST export named functions for shared functionality (parseFrontmatter, loadSubagentFile, discoverSubagents).
- **R-AGENT-007** SHOULD: Use 'fs/promises' instead of callback-based 'fs' for new code to enable async/await patterns and improve error handling.
- **R-AGENT-008** SHOULD: Centralize configuration parsing logic in core utility modules rather than duplicating JSON.parse calls in individual agents.

### Verify

```bash
# Verify path imports in agent and core modules
grep -r "import.*from 'path'" src/agents/ src/core/ | wc -l

# Verify fs/fs-promises imports in agent and core modules
grep -r "import.*from 'fs" src/agents/ src/core/ | wc -l

# Verify exported agent classes
grep -r "export.*class.*Agent" src/agents/ | wc -l

# Verify unprotected JSON.parse calls
grep -r "JSON.parse" src/agents/ src/core/ | grep -v "try" | wc -l
```

**Accept when:**
- All agent implementations in src/agents/ import 'path' from Node.js core
- All agent implementations import 'fs' or 'fs/promises' from Node.js core
- All agent implementations export their class as a public contract
- All JSON.parse calls are wrapped in try-catch blocks or error handling logic
- Core utility modules export named functions for shared functionality (parseFrontmatter, loadSubagentFile, discoverSubagents)
- All agent implementations extend AbstractAgent, AgentsMdAgent, or implement IAgent interface

<enforcement>
Clause Code MUST NOT skip or defer verification. ESLint rules enforcing Node.js core import patterns, code review checklists verifying agent implementations extend base classes or implement interfaces, CI pipeline checks for exported public contracts, and static analysis scanning for unprotected JSON.parse calls are mandatory. Violations result in CI build failure and code review blocks until resolved.
</enforcement>
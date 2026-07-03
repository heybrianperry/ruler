# Standardize Core Library Imports for Agent and Utility Modules: Utility Modules Providing

These rules are ALWAYS ACTIVE for all agent implementations and core utility modules within the codebase, specifically files in `src/agents/` and `src/core/` directories that provide cross-cutting concerns or implement agent contracts.

### Rules

- **R-UTIL-001** SHOULD: Utility modules providing cross-cutting concerns SHOULD export named functions as public contracts (e.g., parseFrontmatter, validateFrontmatter, loadSubagentFile, discoverSubagents).
- **R-UTIL-002** MUST: All agent implementations MUST import Node.js core libraries ('path', 'fs', 'fs/promises') for file system operations.
- **R-UTIL-003** MUST: All agent implementations MUST export their class as a public contract for external consumers.
- **R-UTIL-004** MUST: All JSON.parse calls in agent and utility modules MUST be wrapped in try-catch blocks or equivalent error handling logic.
- **R-UTIL-005** SHOULD: New agent implementations SHOULD extend AbstractAgent or AgentsMdAgent base classes to inherit standard import patterns.
- **R-UTIL-006** SHOULD: New code SHOULD use 'fs/promises' instead of callback-based 'fs' to enable async/await patterns and improve error handling.
- **R-UTIL-007** SHOULD: Configuration parsing logic SHOULD be centralized in core utility modules (config-utils, SubagentsUtils) rather than duplicated in individual agents.

### Verify

```bash
# Verify path imports across agent and core modules
grep -r "import.*from 'path'" src/agents/ src/core/ | wc -l

# Verify fs/fs-promises imports
grep -r "import.*from 'fs" src/agents/ src/core/ | wc -l

# Verify agent class exports
grep -r "export.*class.*Agent" src/agents/ | wc -l

# Verify unprotected JSON.parse calls
grep -r "JSON.parse" src/agents/ src/core/ | grep -v "try" | wc -l
```

**Accept when:**
- All agent implementations in `src/agents/` import 'path' from Node.js core
- All agent implementations export their class as a public contract
- All JSON.parse calls are wrapped in try-catch blocks or error handling logic
- Core utility modules export named functions for shared functionality (parseFrontmatter, loadSubagentFile, discoverSubagents)
- No unprotected JSON.parse calls exist in agent or utility modules

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All R-UTIL rules must be checked before approving changes to agent implementations or core utility modules. Violations must be resolved before merge to main branch.
</enforcement>
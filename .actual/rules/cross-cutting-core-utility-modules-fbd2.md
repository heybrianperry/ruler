# Standardize Core Library Imports for Agent and Utility Modules: Core Utility Modules

These rules are ALWAYS ACTIVE for all agent implementations and core utility modules within the codebase, including all files in `src/agents/` and `src/core/` directories.

### Rules

- **R-CORE-001** SHOULD: Core utility modules SHOULD import 'js-yaml' for YAML configuration parsing and '@iarna/toml' for TOML parsing when handling structured configuration files.
- **R-CORE-002** MUST: All agent implementations MUST import Node.js core libraries ('path', 'fs', 'fs/promises') for file system operations.
- **R-CORE-003** MUST: All agent implementations MUST extend AbstractAgent or implement IAgent interface to establish consistent public contracts.
- **R-CORE-004** MUST: All JSON.parse calls in agent and utility modules MUST be wrapped in try-catch blocks or equivalent error handling logic.
- **R-CORE-005** SHOULD: Core utility modules SHOULD export named functions for shared functionality (parseFrontmatter, loadSubagentFile, discoverSubagents) to enable code reuse.
- **R-CORE-006** SHOULD: New code SHOULD use 'fs/promises' instead of callback-based 'fs' to enable async/await patterns and improve error handling.
- **R-CORE-007** SHOULD: Configuration parsing logic SHOULD be centralized in core utility modules (config-utils, SubagentsUtils) rather than duplicated in individual agents.

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
- Core utility modules that parse configuration files import 'js-yaml' and/or '@iarna/toml' as appropriate

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All agent implementations and core utility modules MUST comply with R-CORE-002, R-CORE-003, and R-CORE-004 before merge. Violations result in CI build failure and code review block.
</enforcement>
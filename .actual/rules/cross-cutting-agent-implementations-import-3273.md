# Standardize Core Library Imports for Agent and Utility Modules: Agent Implementations Import

These rules are ALWAYS ACTIVE for all agent implementation modules in `src/agents/` directory and all core utility modules in `src/core/` directory.

### Rules

- **R-AGENT-IMP-001** MAY: Agent implementations MAY import '../core/FileSystemUtils' for shared file system utility functions

### Verify

```bash
# Verify Node.js core library imports across agent and core modules
grep -r "import.*from 'path'" src/agents/ src/core/ | wc -l
grep -r "import.*from 'fs" src/agents/ src/core/ | wc -l

# Verify agent implementations export public contracts
grep -r "export.*class.*Agent" src/agents/ | wc -l

# Verify JSON.parse calls are protected
grep -r "JSON.parse" src/agents/ src/core/ | grep -v "try" | wc -l
```

**Accept when:**
- All agent implementations in `src/agents/` import 'path' from Node.js core
- All agent implementations export their class as a public contract
- All JSON.parse calls are wrapped in try-catch blocks or error handling logic
- Core utility modules export named functions for shared functionality (parseFrontmatter, loadSubagentFile, discoverSubagents)

<enforcement>
Claude Code MUST NOT skip or defer verification. All four acceptance criteria must be confirmed before approving changes to agent implementations or core utility modules.
</enforcement>
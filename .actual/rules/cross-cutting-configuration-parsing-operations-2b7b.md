# Standardize Core Library Imports for Agent and Utility Modules: Configuration Parsing Operations

These rules are ALWAYS ACTIVE for all agent implementation modules in `src/agents/` directory and all core utility modules in `src/core/` directory, including configuration loading and parsing logic, file system operations for agent discovery, and public API contracts exposed through exported classes and functions.

### Rules

- **R-CONFIG-001** MUST: Configuration parsing operations MUST use JSON.parse with appropriate error handling for security input validation.

### Verify

```bash
# Verify Node.js core library imports in agent and core modules
grep -r "import.*from 'path'" src/agents/ src/core/ | wc -l
grep -r "import.*from 'fs" src/agents/ src/core/ | wc -l

# Verify agent implementations export public contracts
grep -r "export.*class.*Agent" src/agents/ | wc -l

# Verify JSON.parse calls are protected with error handling
grep -r "JSON.parse" src/agents/ src/core/ | grep -v "try" | wc -l
```

**Accept when:**
- All agent implementations in `src/agents/` import `path` from Node.js core
- All agent implementations export their class as a public contract
- All JSON.parse calls are wrapped in try-catch blocks or error handling logic
- Core utility modules export named functions for shared functionality (parseFrontmatter, loadSubagentFile, discoverSubagents)

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse calls must be protected with error handling. Violations must be resolved before merge to main branch. Exceptions require architecture review approval from two senior engineers with documented justification.
</enforcement>
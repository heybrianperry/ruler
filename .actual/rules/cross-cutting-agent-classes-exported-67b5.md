# Standardize Core Library Imports for Agent and Utility Modules: Agent Classes Exported

These rules are ALWAYS ACTIVE for all agent implementation modules in `src/agents/` and core utility modules in `src/core/`, including configuration loading, file system operations, and public API contracts.

### Rules

- **R-AGENT-001** MUST: Agent classes MUST be exported as public contracts to enable external instantiation and configuration.

### Verify

```bash
# Verify Node.js core library imports across agent and utility modules
grep -r "import.*from 'path'" src/agents/ src/core/ | wc -l
grep -r "import.*from 'fs" src/agents/ src/core/ | wc -l

# Verify agent classes are exported
grep -r "export.*class.*Agent" src/agents/ | wc -l

# Verify JSON.parse calls are protected
grep -r "JSON.parse" src/agents/ src/core/ | grep -v "try" | wc -l
```

**Accept when:**
- All agent implementations in `src/agents/` import `path` from Node.js core
- All agent implementations export their class as a public contract
- All JSON.parse calls are wrapped in try-catch blocks or error handling logic
- Core utility modules export named functions for shared functionality (parseFrontmatter, loadSubagentFile, discoverSubagents)

<enforcement>
Claude Code MUST NOT skip or defer verification. All agent classes must be exported as public contracts before code is considered compliant.
</enforcement>
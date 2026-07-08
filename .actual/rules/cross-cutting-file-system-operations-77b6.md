# Standardize Agent Configuration Parsing with Input Validation: File System Operations

These rules are ALWAYS ACTIVE for all agent implementations and configuration processing modules within the codebase, including all files in `src/agents/` and `src/core/apply-engine.ts` that perform TOML and JSON parsing from external file system sources.

### Rules

- **R-FSO-001** SHOULD: File system operations for configuration loading SHOULD validate file paths to prevent directory traversal attacks.

### Verify

```bash
# Check for unprotected parse operations in agent and core modules
grep -r 'JSON\.parse\|parseTOML' src/agents/ src/core/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Verify each agent class has an associated configuration interface or schema
grep -r 'export.*Agent' src/agents/*.ts | xargs -I {} sh -c 'grep -l "interface.*Config" $(dirname {})/$(basename {} .ts).ts'

# Verify apply-engine includes validation logic before applying configurations
find src/agents src/core -name '*.ts' -exec grep -l 'applyRulerConfig\|applyConfigurationsToAgents' {} \; | xargs grep -l 'validate\|schema'
```

**Accept when:**
- All `parseTOML` and `JSON.parse` operations in agent files are wrapped in error handling blocks (try-catch)
- Each agent class (CodexCliAgent, ZedAgent, RooCodeAgent) has an associated configuration interface or schema definition
- The apply-engine module includes validation logic before applying configurations to agents
- File path validation prevents directory traversal attacks in configuration loading operations

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All configuration parsing operations must be protected with input validation and error handling before merge.
</enforcement>
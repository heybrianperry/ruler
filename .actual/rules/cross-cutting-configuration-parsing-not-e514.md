# Standardize Agent Configuration Parsing with Input Validation: Configuration Parsing Not

These rules are ALWAYS ACTIVE for all agent implementations and configuration processing modules within the codebase, including all files in src/agents/ and src/core/ that handle TOML and JSON parsing operations.

### Rules

- **R-CONFIG-001** MUST_NOT: Configuration parsing MUST NOT execute or evaluate code embedded in configuration files.
- **R-CONFIG-002** MUST: All parseTOML and JSON.parse calls in agent files MUST be wrapped in try-catch blocks with specific error types for parse failures vs validation failures.
- **R-CONFIG-003** MUST: Each agent class (CodexCliAgent, ZedAgent, RooCodeAgent) MUST have an associated configuration interface or schema definition.
- **R-CONFIG-004** MUST: The apply-engine module MUST include validation logic before applying configurations to agents via applyConfigurationsToAgents.
- **R-CONFIG-005** SHOULD: Implement a shared validation utility in src/core/ that can be reused across agent implementations to ensure consistency.
- **R-CONFIG-006** SHOULD: Add validation checks in applyConfigurationsToAgents before configuration application to serve as a final security gate.

### Verify

```bash
# Check for unprotected parse operations in agent and core files
grep -r 'JSON\.parse\|parseTOML' src/agents/ src/core/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Verify each agent has a configuration interface
grep -r 'export.*Agent' src/agents/*.ts | xargs -I {} sh -c 'grep -l "interface.*Config" $(dirname {})/$(basename {} .ts).ts'

# Verify validation logic exists in apply-engine and agent files
find src/agents src/core -name '*.ts' -exec grep -l 'applyRulerConfig\|applyConfigurationsToAgents' {} \; | xargs grep -l 'validate\|schema'
```

**Accept when:**
- All parseTOML and JSON.parse operations in agent files are wrapped in error handling blocks
- Each agent class (CodexCliAgent, ZedAgent, RooCodeAgent) has an associated configuration interface or schema definition
- The apply-engine module includes validation logic before applying configurations to agents
- Configuration validation occurs at parse boundaries and before concurrent application to prevent race conditions with invalid data

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration parsing operations MUST be validated before use. CI pipeline MUST fail if unprotected parse operations are detected in agent or configuration modules.
</enforcement>
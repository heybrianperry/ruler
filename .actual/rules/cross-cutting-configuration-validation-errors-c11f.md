# Standardize Agent Configuration Parsing with Input Validation: Configuration Validation Errors

These rules are ALWAYS ACTIVE for all agent implementations and configuration processing modules within the codebase, including all files in src/agents/ directory (CodexCliAgent.ts, ZedAgent.ts, RooCodeAgent.ts), configuration processing in src/core/apply-engine.ts, and all TOML and JSON parsing operations using @iarna/toml and JSON.parse.

### Rules

- **R-CONFIG-001** SHOULD: Configuration validation errors SHOULD be logged with sufficient detail for debugging while avoiding exposure of sensitive system information.

### Verify

```bash
# Verify all parseTOML and JSON.parse operations are wrapped in error handling
grep -r 'JSON\.parse\|parseTOML' src/agents/ src/core/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Verify each agent class has an associated configuration interface or schema definition
grep -r 'export.*Agent' src/agents/*.ts | xargs -I {} sh -c 'grep -l "interface.*Config" $(dirname {})/$(basename {} .ts).ts'

# Verify apply-engine module includes validation logic before applying configurations
find src/agents src/core -name '*.ts' -exec grep -l 'applyRulerConfig\|applyConfigurationsToAgents' {} \; | xargs grep -l 'validate\|schema'
```

**Accept when:**
- All parseTOML and JSON.parse operations in agent files are wrapped in error handling blocks
- Each agent class (CodexCliAgent, ZedAgent, RooCodeAgent) has an associated configuration interface or schema definition
- The apply-engine module includes validation logic before applying configurations to agents
- Configuration validation errors are logged with sufficient detail for debugging without exposing sensitive system information

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration parsing operations must include proper error handling and validation before being merged into the codebase.
</enforcement>
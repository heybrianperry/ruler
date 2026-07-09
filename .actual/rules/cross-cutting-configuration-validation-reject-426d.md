# Standardize Agent Configuration Parsing with Input Validation: Configuration Validation Reject

These rules are ALWAYS ACTIVE for all agent implementations and configuration processing modules within the codebase, including all files in src/agents/ directory (CodexCliAgent.ts, ZedAgent.ts, RooCodeAgent.ts), configuration processing in src/core/apply-engine.ts, and all TOML and JSON parsing operations.

### Rules

- **R-CONFIG-001** SHOULD: Configuration validation SHOULD reject unknown or unexpected fields to prevent configuration injection attacks.

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
- Configuration validation rejects unknown or unexpected fields at parse boundaries
- Validation occurs before concurrent application to prevent race conditions with invalid data

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration parsing operations MUST include validation that rejects unknown fields. Violations detected in CI pipeline MUST block merge. Security team MUST be notified of validation bypasses discovered in production code.
</enforcement>
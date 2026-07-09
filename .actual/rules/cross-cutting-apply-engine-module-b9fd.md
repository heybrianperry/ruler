# Standardize Agent Configuration Parsing with Input Validation: Apply Engine Module

These rules are ALWAYS ACTIVE for all agent implementations and configuration processing modules within the codebase, including all files in src/agents/ and src/core/ that handle TOML and JSON parsing operations.

### Rules

- **R-APPLY-001** MUST: The apply-engine module MUST validate hierarchical configurations before applying them through applyConfigurationsToAgents.
- **R-APPLY-002** MUST: All parseTOML and JSON.parse operations in agent files (CodexCliAgent.ts, ZedAgent.ts, RooCodeAgent.ts) MUST be wrapped in try-catch blocks with specific error types for parse failures vs validation failures.
- **R-APPLY-003** MUST: Each agent class (CodexCliAgent, ZedAgent, RooCodeAgent) MUST have an associated configuration interface or schema definition.
- **R-APPLY-004** MUST: Configuration validation MUST occur at parse boundaries before concurrent application to prevent race conditions with invalid data.
- **R-APPLY-005** MUST: Unknown configuration fields MUST be rejected by default (deny-by-default policy).
- **R-APPLY-006** SHOULD: Implement a shared validation utility in src/core/ that can be reused across agent implementations to ensure consistency.
- **R-APPLY-007** SHOULD: Use TypeScript type guards or assertion functions to provide compile-time and runtime type safety.

### Verify

```bash
# Check for unprotected parse operations in agent and core files
grep -r 'JSON\.parse\|parseTOML' src/agents/ src/core/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Verify each agent has a configuration interface
grep -r 'export.*Agent' src/agents/*.ts | xargs -I {} sh -c 'grep -l "interface.*Config" $(dirname {})/$(basename {} .ts).ts'

# Verify apply-engine includes validation logic
find src/agents src/core -name '*.ts' -exec grep -l 'applyRulerConfig\|applyConfigurationsToAgents' {} \; | xargs grep -l 'validate\|schema'
```

**Accept when:**
- All parseTOML and JSON.parse operations in agent files are wrapped in error handling blocks
- Each agent class (CodexCliAgent, ZedAgent, RooCodeAgent) has an associated configuration interface or schema definition
- The apply-engine module includes validation logic before applying configurations to agents
- Configuration validation occurs before concurrent application through applyConfigurationsToAgents
- Unknown configuration fields are rejected by default

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for any code changes affecting agent configuration parsing, TOML/JSON loading, or the apply-engine module.
</enforcement>
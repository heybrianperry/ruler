# Standardize Agent Configuration Parsing with Input Validation: Agent Public Contracts

These rules are ALWAYS ACTIVE for all agent implementations and configuration processing modules within the codebase, specifically targeting configuration parsing from external TOML and JSON files in agent public contracts (CodexCliAgent, ZedAgent, RooCodeAgent) and the apply-engine module.

### Rules

- **R-AGENT-001** MUST: Agent public contracts (CodexCliAgent, ZedAgent, RooCodeAgent) MUST define explicit configuration schemas that specify allowed fields, types, and value constraints.
- **R-AGENT-002** MUST: All parseTOML and JSON.parse operations in agent files (src/agents/) and configuration processing modules (src/core/apply-engine.ts) MUST be wrapped in try-catch blocks with specific error types for parse failures vs validation failures.
- **R-AGENT-003** MUST: Configuration validation MUST occur at agent public contract boundaries before configuration application to prevent injection attacks and malformed data handling.
- **R-AGENT-004** MUST: The apply-engine module MUST include validation logic in applyConfigurationsToAgents before applying configurations to agents, serving as a final security gate.
- **R-AGENT-005** SHOULD: Implement a shared validation utility in src/core/ that can be reused across agent implementations to ensure consistency.
- **R-AGENT-006** SHOULD: Use TypeScript interfaces for RulerConfiguration and HierarchicalRulerConfiguration that serve as validation schemas.
- **R-AGENT-007** SHOULD: Implement deny-by-default validation for unknown configuration fields to prevent exploitation through undocumented fields.

### Verify

```bash
# Check for unprotected parse operations in agent and core modules
grep -r 'JSON\.parse\|parseTOML' src/agents/ src/core/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Verify each agent class has an associated configuration interface or schema
grep -r 'export.*Agent' src/agents/*.ts | xargs -I {} sh -c 'grep -l "interface.*Config" $(dirname {})/$(basename {} .ts).ts'

# Find files with configuration parsing and verify they include validation logic
find src/agents src/core -name '*.ts' -exec grep -l 'applyRulerConfig\|applyConfigurationsToAgents' {} \; | xargs grep -l 'validate\|schema'
```

**Accept when:**
- All parseTOML and JSON.parse operations in agent files are wrapped in error handling blocks
- Each agent class (CodexCliAgent, ZedAgent, RooCodeAgent) has an associated configuration interface or schema definition
- The apply-engine module includes validation logic before applying configurations to agents
- Configuration validation occurs at agent public contract boundaries before concurrent application
- Unknown configuration fields are rejected by default

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration parsing operations must be protected with validation before merging. Security team must review validation schemas for completeness.
</enforcement>
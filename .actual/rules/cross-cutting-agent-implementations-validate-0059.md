# Standardize Agent Configuration Parsing with Input Validation: Agent Implementations Validate

These rules are ALWAYS ACTIVE for all agent implementations and configuration processing modules within the codebase, including all files in `src/agents/` directory (CodexCliAgent.ts, ZedAgent.ts, RooCodeAgent.ts), configuration processing in `src/core/apply-engine.ts`, and all TOML and JSON parsing operations using @iarna/toml and JSON.parse.

### Rules

- **R-AGENT-CONFIG-001** MUST: All agent implementations MUST validate configuration input immediately after parsing TOML or JSON content using parseTOML or JSON.parse, before any further processing or application to agent instances.

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
- All parseTOML and JSON.parse operations in agent files are wrapped in error handling blocks (try-catch)
- Each agent class (CodexCliAgent, ZedAgent, RooCodeAgent) has an associated configuration interface or schema definition
- The apply-engine module includes validation logic before applying configurations to agents
- Configuration validation occurs at parse boundaries, not deferred to later stages
- Validation rejects malformed and malicious configuration inputs with specific error types

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All parseTOML and JSON.parse operations must be protected with validation logic before merging any configuration-related code.
</enforcement>
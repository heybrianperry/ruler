# Standardize Core Library Imports for Agent and Configuration Modules: Agent Implementations Import

These rules are ALWAYS ACTIVE for all agent implementation modules in `src/agents/`, configuration loading modules (UnifiedConfigLoader, ConfigLoader), core utility modules supporting agent operations (revert-engine, FileSystemUtils), and modules that parse or serialize TOML or JSON configuration data.

### Rules

- **R-AGENT-IMP-001** MAY: Agent implementations MAY import additional core utilities (hash, merge, agent-utils) as needed for specific functionality.
- **R-AGENT-IMP-002** MUST: All agent implementations MUST import from IAgent or AgentsMdAgent interfaces to establish consistent public contracts.
- **R-AGENT-IMP-003** MUST: TOML parsing operations MUST consistently use @iarna/toml library across all configuration loading paths.
- **R-AGENT-IMP-004** MUST: JSON parsing MUST use native JSON.parse rather than alternative JSON parsing libraries.
- **R-AGENT-IMP-005** MUST: Filesystem operations MUST be centralized through FileSystemUtils or native Node.js modules to reduce duplication.
- **R-AGENT-IMP-006** SHOULD: Agent implementations SHOULD apply configuration through the applyRulerConfig concurrency model pattern.
- **R-AGENT-IMP-EX-001** EXCEPTION: Legacy agent implementations are granted a migration period before adopting standardized imports, with documented exception and expiration review date.

### Verify

```bash
# Verify IAgent/AgentsMdAgent imports across agent implementations
grep -r "from ['\"].*IAgent['\"]" src/agents/*.ts | wc -l

# Verify @iarna/toml usage for TOML parsing
grep -r "@iarna/toml" src/ --include="*.ts" | grep -v node_modules

# Verify JSON.parse usage in configuration contexts
grep -r "JSON.parse" src/agents/*.ts src/core/*.ts | grep -E "(config|Config|toml|json)"

# Verify agent implementations export public contracts
find src/agents -name "*.ts" -exec grep -l "export.*Agent" {} \;

# Verify no alternative filesystem or parsing libraries without exception
grep -r "require.*fs-extra\|import.*fs-extra" src/agents/ --include="*.ts"
grep -r "require.*json5\|import.*json5" src/agents/ --include="*.ts"
```

**Accept when:**
- All agent implementation files in `src/agents/` import from IAgent or AgentsMdAgent interfaces
- TOML parsing operations consistently use @iarna/toml library
- JSON parsing uses native JSON.parse in configuration contexts
- Agent modules export public contracts
- No agent implementation uses alternative filesystem or parsing libraries without documented exception
- Any deviations from standard import patterns are documented with ADR reference and expiration review date

<enforcement>
Claude Code MUST NOT skip or defer verification. All agent implementation files MUST be audited for compliance with these import standards before merge. CI pipeline checks using grep patterns MUST detect non-standard library usage and fail the build unless an approved exception is documented.
</enforcement>
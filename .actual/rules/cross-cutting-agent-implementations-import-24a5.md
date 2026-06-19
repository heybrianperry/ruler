# Standardize Core Library Imports for Agent and Configuration Modules: Agent Implementations Import

These rules are ALWAYS ACTIVE for all agent implementations and configuration loading modules within the codebase, including all files in src/agents/, configuration loading modules (UnifiedConfigLoader, ConfigLoader), core utility modules (revert-engine, FileSystemUtils), and modules that parse or serialize TOML or JSON configuration data.

### Rules

- **R-AGENT-IMP-001** SHOULD: Agent implementations SHOULD import from AgentsMdAgent when extending base agent functionality.

### Verify

```bash
# Verify IAgent imports are present in agent implementations
grep -r "from ['\"].*IAgent['\"]" src/agents/*.ts | wc -l

# Verify @iarna/toml is used for TOML parsing
grep -r "@iarna/toml" src/ --include="*.ts" | grep -v node_modules

# Verify JSON.parse is used for configuration parsing
grep -r "JSON.parse" src/agents/*.ts src/core/*.ts | grep -E "(config|Config|toml|json)"

# Verify agent implementations export public contracts
find src/agents -name "*.ts" -exec grep -l "export.*Agent" {} \;
```

**Accept when:**
- All agent implementation files in src/agents/ import from IAgent or AgentsMdAgent interfaces
- TOML parsing operations consistently use @iarna/toml library and JSON parsing uses native JSON.parse
- Agent modules export public contracts and no agent implementation uses alternative filesystem or parsing libraries without documented exception
- No non-standard imports are detected in agent implementations without documented exceptions

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via ESLint rules, code review checklists, and CI pipeline checks using grep patterns are mandatory. Violations block CI builds and code review merges until imports conform to standard patterns or approved exceptions are documented.
</enforcement>
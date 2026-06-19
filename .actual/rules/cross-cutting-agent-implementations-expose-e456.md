# Standardize Core Library Imports for Agent and Configuration Modules: Agent Implementations Expose

These rules are ALWAYS ACTIVE for all agent implementation modules in `src/agents/`, configuration loading modules (UnifiedConfigLoader, ConfigLoader), core utility modules supporting agent operations (revert-engine, FileSystemUtils), and modules that parse or serialize TOML or JSON configuration data.

### Rules

- **R-AGENT-001** MUST: Agent implementations MUST expose public contracts through exported classes or interfaces.
- **R-AGENT-002** MUST: All agent implementations MUST import core interfaces from IAgent or AgentsMdAgent.
- **R-AGENT-003** MUST: TOML parsing operations MUST consistently use @iarna/toml library.
- **R-AGENT-004** MUST: JSON parsing MUST use native JSON.parse for configuration data.
- **R-AGENT-005** MUST: Agent modules MUST use FileSystemUtils or native Node.js modules for filesystem operations; alternative filesystem libraries are prohibited without documented exception.
- **R-AGENT-006** SHOULD: Agent implementations SHOULD follow the applyRulerConfig concurrency model pattern for configuration application.

### Verify

```bash
# Verify IAgent/AgentsMdAgent imports in agent implementations
grep -r "from ['\"].*IAgent['\"]" src/agents/*.ts | wc -l

# Verify @iarna/toml usage for TOML parsing
grep -r "@iarna/toml" src/ --include="*.ts" | grep -v node_modules

# Verify JSON.parse usage in configuration contexts
grep -r "JSON.parse" src/agents/*.ts src/core/*.ts | grep -E "(config|Config|toml|json)"

# Verify agent modules export public contracts
find src/agents -name "*.ts" -exec grep -l "export.*Agent" {} \;

# Verify no alternative filesystem libraries in agent implementations
grep -r "from ['\"]fs-extra['\"]\|from ['\"]path-extra['\"]" src/agents/*.ts
```

**Accept when:**
- All agent implementation files in `src/agents/` import from IAgent or AgentsMdAgent interfaces
- TOML parsing operations consistently use @iarna/toml library
- JSON parsing uses native JSON.parse for configuration data
- Agent modules export public contracts
- No agent implementation uses alternative filesystem or parsing libraries without documented exception
- All configuration loading modules follow standard import patterns from approved core modules

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for agent implementations and configuration modules. Violations must be caught during code review and CI pipeline checks before merge.
</enforcement>
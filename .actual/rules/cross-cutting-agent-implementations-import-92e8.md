# Standardize Core Library Imports for Agent and Configuration Modules: Agent Implementations Import

These rules are ALWAYS ACTIVE for all agent implementation modules in `src/agents/`, configuration loading modules (UnifiedConfigLoader, ConfigLoader), core utility modules that support agent operations (revert-engine, FileSystemUtils), and modules that parse or serialize TOML or JSON configuration data.

### Rules

- **R-AGENT-IMP-001** MUST: Agent implementations MUST import from the IAgent interface to establish the base contract.
- **R-AGENT-IMP-002** MUST: TOML parsing operations MUST consistently use @iarna/toml library.
- **R-AGENT-IMP-003** MUST: JSON parsing MUST use native JSON.parse.
- **R-AGENT-IMP-004** MUST: Agent modules MUST export public contracts and apply configuration through the applyRulerConfig concurrency model pattern.
- **R-AGENT-IMP-005** MUST: Filesystem operations MUST be centralized through FileSystemUtils and native Node.js modules.

### Verify

```bash
# Verify IAgent imports across agent implementations
grep -r "from ['\"].*IAgent['\"]" src/agents/*.ts | wc -l

# Verify @iarna/toml usage for TOML parsing
grep -r "@iarna/toml" src/ --include="*.ts" | grep -v node_modules

# Verify JSON.parse usage in configuration contexts
grep -r "JSON.parse" src/agents/*.ts src/core/*.ts | grep -E "(config|Config|toml|json)"

# Find all agent implementation exports
find src/agents -name "*.ts" -exec grep -l "export.*Agent" {} \;
```

**Accept when:**
- All agent implementation files in `src/agents/` import from IAgent or AgentsMdAgent interfaces
- TOML parsing operations consistently use @iarna/toml library and JSON parsing uses native JSON.parse
- Agent modules export public contracts and no agent implementation uses alternative filesystem or parsing libraries without documented exception
- Configuration loading modules (UnifiedConfigLoader, ConfigLoader) follow the same import standardization patterns

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via ESLint rules, code review checklists, and CI pipeline checks using grep patterns are mandatory. Violations block CI builds and code review merges unless documented exceptions are approved by the architecture review board.
</enforcement>
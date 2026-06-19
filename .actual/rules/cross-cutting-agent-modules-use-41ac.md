# Standardize Core Library Imports for Agent and Configuration Modules: Agent Modules Use

These rules are ALWAYS ACTIVE for all agent implementation modules in `src/agents/`, configuration loading modules (UnifiedConfigLoader, ConfigLoader), core utility modules that support agent operations (revert-engine, FileSystemUtils), and modules that parse or serialize TOML or JSON configuration data.

### Rules

- **R-AGENT-001** MUST: Agent modules MUST use native Node.js modules (fs, fs/promises, path) for filesystem operations rather than third-party alternatives.

### Verify

```bash
# Verify IAgent imports are present in agent implementations
grep -r "from ['\"].*IAgent['\"]" src/agents/*.ts | wc -l

# Verify @iarna/toml is used for TOML parsing
grep -r "@iarna/toml" src/ --include="*.ts" | grep -v node_modules

# Verify JSON.parse is used for JSON configuration
grep -r "JSON.parse" src/agents/*.ts src/core/*.ts | grep -E "(config|Config|toml|json)"

# Find all agent implementations
find src/agents -name "*.ts" -exec grep -l "export.*Agent" {} \;
```

**Accept when:**
- All agent implementation files in `src/agents/` import from IAgent or AgentsMdAgent interfaces
- TOML parsing operations consistently use @iarna/toml library and JSON parsing uses native JSON.parse
- Agent modules export public contracts and no agent implementation uses alternative filesystem or parsing libraries without documented exception
- No non-standard imports are detected in configuration loading modules or core utility modules supporting agent operations

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via ESLint rules, code review checklists, and CI pipeline checks using grep patterns are mandatory. Violations block CI builds and code review merges unless documented exceptions are approved by the architecture review board.
</enforcement>
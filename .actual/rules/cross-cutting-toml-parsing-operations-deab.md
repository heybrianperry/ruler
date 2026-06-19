# Standardize Core Library Imports for Agent and Configuration Modules: Toml Parsing Operations

These rules are ALWAYS ACTIVE for all agent implementation modules in `src/agents/`, configuration loading modules (UnifiedConfigLoader, ConfigLoader), core utility modules that support agent operations (revert-engine, FileSystemUtils), and any modules that parse or serialize TOML or JSON configuration data.

### Rules

- **R-TOML-001** MUST: TOML parsing operations MUST use the @iarna/toml library via parseTOML function calls.

### Verify

```bash
# Verify IAgent imports are present in agent implementations
grep -r "from ['\"].*IAgent['\"]" src/agents/*.ts | wc -l

# Verify @iarna/toml usage across the codebase
grep -r "@iarna/toml" src/ --include="*.ts" | grep -v node_modules

# Verify JSON.parse usage in configuration contexts
grep -r "JSON.parse" src/agents/*.ts src/core/*.ts | grep -E "(config|Config|toml|json)"

# Find all agent implementations
find src/agents -name "*.ts" -exec grep -l "export.*Agent" {} \;
```

**Accept when:**
- All agent implementation files in `src/agents/` import from IAgent or AgentsMdAgent interfaces
- TOML parsing operations consistently use @iarna/toml library and JSON parsing uses native JSON.parse
- Agent modules export public contracts and no agent implementation uses alternative filesystem or parsing libraries without documented exception
- No non-standard TOML parsing libraries are detected in configuration loading modules

<enforcement>
Claude Code MUST NOT skip or defer verification. All TOML parsing operations must be audited against the @iarna/toml standard before code review approval.
</enforcement>
# Standardize Core Library Imports for Agent and Configuration Modules: Configuration Management Modules

These rules are ALWAYS ACTIVE for all agent implementation modules in src/agents/, configuration loading modules (UnifiedConfigLoader, ConfigLoader), core utility modules that support agent operations (revert-engine, FileSystemUtils), and modules that parse or serialize TOML or JSON configuration data.

### Rules

- **R-CONFIG-001** SHOULD: Configuration management modules SHOULD coordinate through UnifiedConfigLoader and RuleProcessor for consistent configuration handling.

### Verify

```bash
# Verify IAgent imports are used consistently
grep -r "from ['\"].*IAgent['\"]" src/agents/*.ts | wc -l

# Verify @iarna/toml is used for TOML parsing
grep -r "@iarna/toml" src/ --include="*.ts" | grep -v node_modules

# Verify JSON.parse is used for JSON configuration
grep -r "JSON.parse" src/agents/*.ts src/core/*.ts | grep -E "(config|Config|toml|json)"

# Verify agent implementations export public contracts
find src/agents -name "*.ts" -exec grep -l "export.*Agent" {} \;
```

**Accept when:**
- All agent implementation files in src/agents/ import from IAgent or AgentsMdAgent interfaces
- TOML parsing operations consistently use @iarna/toml library and JSON parsing uses native JSON.parse
- Agent modules export public contracts and no agent implementation uses alternative filesystem or parsing libraries without documented exception
- Configuration management modules coordinate through UnifiedConfigLoader and RuleProcessor

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via ESLint rules checking import statements against approved core module list, code review checklist verification, and CI pipeline checks using grep patterns are mandatory. Violations result in CI build failure or code review block until imports conform to standard patterns or documented exception is approved.
</enforcement>
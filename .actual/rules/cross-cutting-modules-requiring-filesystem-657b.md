# Standardize Core Library Imports for Agent and Configuration Modules: Modules Requiring Filesystem

These rules are ALWAYS ACTIVE for all agent implementation modules in src/agents/, configuration loading modules (UnifiedConfigLoader, ConfigLoader), core utility modules that support agent operations (revert-engine, FileSystemUtils), and modules that parse or serialize TOML or JSON configuration data.

### Rules

- **R-CORE-001** SHOULD: Modules requiring filesystem utilities SHOULD import from ../core/FileSystemUtils rather than implementing custom file operations.

### Verify

```bash
# Verify IAgent imports are consistent across agent implementations
grep -r "from ['\"].*IAgent['\"]" src/agents/*.ts | wc -l

# Verify @iarna/toml usage for TOML parsing
grep -r "@iarna/toml" src/ --include="*.ts" | grep -v node_modules

# Verify JSON.parse usage in configuration contexts
grep -r "JSON.parse" src/agents/*.ts src/core/*.ts | grep -E "(config|Config|toml|json)"

# Find all agent implementations
find src/agents -name "*.ts" -exec grep -l "export.*Agent" {} \;
```

**Accept when:**
- All agent implementation files in src/agents/ import from IAgent or AgentsMdAgent interfaces
- TOML parsing operations consistently use @iarna/toml library and JSON parsing uses native JSON.parse
- Agent modules export public contracts and no agent implementation uses alternative filesystem or parsing libraries without documented exception
- FileSystemUtils is the single point of import for filesystem operations across all in-scope modules

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via ESLint rules, code review checklists, and CI pipeline checks using grep patterns are mandatory. Violations block CI builds and code review merges unless documented exceptions are approved by the architecture review board.
</enforcement>
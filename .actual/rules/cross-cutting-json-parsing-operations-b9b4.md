# Standardize Core Library Imports for Agent and Configuration Modules: Json Parsing Operations

These rules are ALWAYS ACTIVE for all agent implementation modules in `src/agents/`, configuration loading modules (UnifiedConfigLoader, ConfigLoader), core utility modules that support agent operations (revert-engine, FileSystemUtils), and modules that parse or serialize TOML or JSON configuration data.

### Rules

- **R-JSON-001** MUST: JSON parsing operations MUST use native `JSON.parse` for configuration and data deserialization.

### Verify

```bash
# Verify JSON.parse usage in agent and configuration modules
grep -r "JSON.parse" src/agents/*.ts src/core/*.ts | grep -E "(config|Config|toml|json)" | wc -l

# Verify no alternative JSON parsing libraries are used
grep -r "JSON\.stringify\|jsonparse\|fast-json-parse" src/agents/ src/core/ --include="*.ts" | grep -v node_modules | wc -l

# Verify @iarna/toml is used for TOML parsing
grep -r "@iarna/toml" src/ --include="*.ts" | grep -v node_modules | wc -l

# Verify agent implementations export public contracts
find src/agents -name "*.ts" -exec grep -l "export.*Agent" {} \;
```

**Accept when:**
- All JSON parsing operations in agent and configuration modules use native `JSON.parse`
- No alternative JSON parsing libraries (jsonparse, fast-json-parse, etc.) are detected in agent or core modules
- TOML parsing consistently uses `@iarna/toml` library
- All agent implementation files export public contracts
- No agent implementation uses alternative parsing libraries without documented exception (EX-001)

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON parsing operations must conform to R-JSON-001 or have an approved exception documented in code comments with ADR reference.
</enforcement>
# Validate JSON Input Before Parsing in Configuration and Agent Modules: Configuration Modules Validate

These rules are ALWAYS ACTIVE for all agent and configuration loading modules that parse external JSON or TOML content from file system sources and environment variables.

### Rules

- **R-CONFIG-001** MUST: Configuration modules MUST validate parsed content using Zod schemas (z.object, z.string, z.boolean, z.array, z.record) before accepting configuration values.
- **R-CONFIG-002** MUST: Wrap all JSON.parse and TOML.parse operations in try-catch blocks with console.warn logging that includes file path and error details.
- **R-CONFIG-003** MUST: Define Zod schemas for all configuration objects using z.object() with appropriate field types (z.string, z.boolean, z.array, z.record, z.enum).
- **R-CONFIG-004** SHOULD: Use Zod's .optional() modifier for non-required configuration fields to support partial configurations.
- **R-CONFIG-005** SHOULD: Return null or default configuration objects when parsing fails to enable graceful degradation.
- **R-CONFIG-006** SHOULD: Include module-specific prefixes in log messages (e.g., '[ruler]') to facilitate log filtering and debugging.

### Verify

```bash
# Verify all JSON.parse calls in configuration modules are wrapped in try-catch
grep -r 'JSON\.parse' src/ | grep -v 'try\|catch' || echo 'All JSON.parse calls are wrapped'

# Verify console.warn logging is present in configuration modules
grep -r 'console\.warn' src/agents/FirebenderAgent.ts src/core/ConfigLoader.ts | grep -c 'Failed\|Warning'

# Verify Zod schema validation is in use
grep -r 'z\.object\|z\.string\|z\.boolean' src/core/ConfigLoader.ts | wc -l
```

**Accept when:**
- All JSON.parse operations in configuration loading code are wrapped in try-catch blocks with error logging
- ConfigLoader uses Zod schema validation for all configuration objects before accepting values
- Parse failures generate console.warn log entries with file path and error message context
- Zod schemas are defined for FirebenderAgent configuration, TOML config files, agent configuration, and MCP configuration
- Module-specific prefixes appear in log messages for configuration loading failures

<enforcement>
Clause Code MUST NOT skip or defer verification. All configuration parsing code must demonstrate compliance with R-CONFIG-001 through R-CONFIG-006 before acceptance.
</enforcement>
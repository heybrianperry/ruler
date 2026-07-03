# Validate JSON Input Before Parsing in Configuration and Agent Modules: Log Messages Include

These rules are ALWAYS ACTIVE for all agent and configuration loading modules that parse external JSON or TOML content from file system sources and environment variables.

### Rules

- **R-CONFIG-001** MUST: Wrap all JSON.parse and TOML.parse operations in try-catch blocks with console.warn logging that includes file path and error details.
- **R-CONFIG-002** MUST: Define Zod schemas for all configuration objects using z.object() with appropriate field types (z.string, z.boolean, z.array, z.record, z.enum).
- **R-CONFIG-003** MUST: Use Zod's .optional() modifier for non-required configuration fields to support partial configurations.
- **R-CONFIG-004** MUST: Return null or default configuration objects when parsing fails to enable graceful degradation.
- **R-CONFIG-005** SHOULD: Log messages SHOULD include a prefix identifying the module or component (e.g., '[ruler]') for log filtering and analysis.
- **R-CONFIG-006** SHOULD: Validate configuration once during loading and cache validated results; avoid re-parsing on every access.

### Verify

```bash
# Verify all JSON.parse calls are wrapped in try-catch blocks
grep -r 'JSON\.parse' src/ | grep -v 'try\|catch' || echo 'All JSON.parse calls are wrapped'

# Verify console.warn logging is present in configuration modules
grep -r 'console\.warn' src/agents/FirebenderAgent.ts src/core/ConfigLoader.ts | grep -c 'Failed\|Warning'

# Verify Zod schema validation is in place
grep -r 'z\.object\|z\.string\|z\.boolean' src/core/ConfigLoader.ts | wc -l

# Verify module prefixes in log messages
grep -r 'console\.warn' src/agents/FirebenderAgent.ts src/core/ConfigLoader.ts | grep -c '\[.*\]'
```

**Accept when:**
- All JSON.parse operations in configuration loading code are wrapped in try-catch blocks with error logging
- ConfigLoader uses Zod schema validation for all configuration objects before accepting values
- Parse failures generate console.warn log entries with file path and error message context
- Log messages include module-specific prefixes (e.g., '[ruler]') for filtering and debugging
- Configuration validation occurs once at load time with results cached for subsequent access

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for configuration and agent module code paths.
</enforcement>
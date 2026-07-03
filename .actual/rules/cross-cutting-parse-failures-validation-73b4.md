# Validate JSON Input Before Parsing in Configuration and Agent Modules: Parse Failures Validation

These rules are ALWAYS ACTIVE for all agent and configuration loading modules that parse external JSON or TOML content from file system sources and environment variables.

### Rules

- **R-PARSE-001** MUST: Parse failures and validation errors MUST be logged using console.warn with contextual information including file path and error message.
- **R-PARSE-002** MUST: Wrap all JSON.parse and TOML.parse operations in try-catch blocks with console.warn logging that includes file path and error details.
- **R-PARSE-003** MUST: Define Zod schemas for all configuration objects using z.object() with appropriate field types (z.string, z.boolean, z.array, z.record, z.enum).
- **R-PARSE-004** SHOULD: Use Zod's .optional() modifier for non-required configuration fields to support partial configurations.
- **R-PARSE-005** SHOULD: Return null or default configuration objects when parsing fails to enable graceful degradation.
- **R-PARSE-006** SHOULD: Include module-specific prefixes in log messages (e.g., '[ruler]') to facilitate log filtering and debugging.

### Verify

```bash
# Verify all JSON.parse calls are wrapped in try-catch
grep -r 'JSON\.parse' src/ | grep -v 'try\|catch' || echo 'All JSON.parse calls are wrapped'

# Verify console.warn logging in configuration modules
grep -r 'console\.warn' src/agents/FirebenderAgent.ts src/core/ConfigLoader.ts | grep -c 'Failed\|Warning'

# Verify Zod schema usage
grep -r 'z\.object\|z\.string\|z\.boolean' src/core/ConfigLoader.ts | wc -l
```

**Accept when:**
- All JSON.parse operations in configuration loading code are wrapped in try-catch blocks with error logging
- ConfigLoader uses Zod schema validation for all configuration objects before accepting values
- Parse failures generate console.warn log entries with file path and error message context
- FirebenderAgent configuration loading (firebender.json) implements safe parsing with validation
- ConfigLoader TOML and JSON parsing includes error handling and logging
- Agent configuration schema validation uses Zod with appropriate field types
- MCP configuration parsing includes try-catch and console.warn logging
- Environment variable processing (process.env.XDG_CONFIG_HOME) is validated before use

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. All JSON.parse and TOML.parse operations in configuration modules MUST be wrapped with try-catch and console.warn logging. Zod schema validation MUST be applied to all configuration objects before use.
</enforcement>
# Validate JSON Input Before Parsing in Configuration and Agent Modules: Modules Use Additional

These rules are ALWAYS ACTIVE for all agent and configuration loading modules that parse external JSON or TOML content from file system sources and environment variables.

### Rules

- **R-CONFIG-001** MUST: Wrap all JSON.parse and TOML.parse operations in try-catch blocks with console.warn logging that includes file path and error details.
- **R-CONFIG-002** MUST: Define Zod schemas for all configuration objects using z.object() with appropriate field types (z.string, z.boolean, z.array, z.record, z.enum).
- **R-CONFIG-003** MUST: Use Zod's .optional() modifier for non-required configuration fields to support partial configurations.
- **R-CONFIG-004** MUST: Return null or default configuration objects when parsing fails to enable graceful degradation.
- **R-CONFIG-005** MUST: Include module-specific prefixes in log messages (e.g., '[ruler]') to facilitate log filtering and debugging.
- **R-CONFIG-006** MAY: Use additional validation libraries beyond Zod if they provide equivalent schema-based validation capabilities.

### Verify

```bash
# Verify all JSON.parse calls are wrapped in try-catch
grep -r 'JSON\.parse' src/ | grep -v 'try\|catch' || echo 'All JSON.parse calls are wrapped'

# Verify console.warn logging is present in configuration modules
grep -r 'console\.warn' src/agents/FirebenderAgent.ts src/core/ConfigLoader.ts | grep -c 'Failed\|Warning'

# Verify Zod schema validation is in place
grep -r 'z\.object\|z\.string\|z\.boolean' src/core/ConfigLoader.ts | wc -l
```

**Accept when:**
- All JSON.parse operations in configuration loading code are wrapped in try-catch blocks with error logging
- ConfigLoader uses Zod schema validation for all configuration objects before accepting values
- Parse failures generate console.warn log entries with file path and error message context
- Module-specific prefixes are present in all configuration-related log messages
- Non-required configuration fields use Zod's .optional() modifier
- Configuration parsing failures return null or default objects rather than throwing exceptions

<enforcement>
Clause Code MUST NOT skip or defer verification. All JSON.parse and TOML.parse operations in configuration modules MUST be wrapped with try-catch and logging. Zod schema validation MUST be applied to all configuration objects. Pull requests with unwrapped parse calls or missing validation are rejected. Runtime exceptions from configuration parsing are treated as P1 bugs.
</enforcement>
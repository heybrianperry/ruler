# Validate JSON Input Before Parsing in Configuration and Agent Modules: Json Parse Operations

These rules are ALWAYS ACTIVE for all agent and configuration loading modules that parse external JSON or TOML content from file system sources and environment variables.

### Rules

- **R-JSON-001** MUST: All JSON.parse() operations on external file content MUST be wrapped in try-catch blocks to handle parsing exceptions.
- **R-JSON-002** MUST: All TOML.parse() operations on external file content MUST be wrapped in try-catch blocks to handle parsing exceptions.
- **R-JSON-003** MUST: Define Zod schemas for all configuration objects using z.object() with appropriate field types (z.string, z.boolean, z.array, z.record, z.enum).
- **R-JSON-004** MUST: Use Zod's .optional() modifier for non-required configuration fields to support partial configurations.
- **R-JSON-005** MUST: Return null or default configuration objects when parsing fails to enable graceful degradation.
- **R-JSON-006** MUST: Include module-specific prefixes in log messages (e.g., '[ruler]') when logging parse failures via console.warn.
- **R-JSON-007** SHOULD: Include file path and error details in console.warn log messages for parse failures.

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
- All TOML.parse operations in configuration loading code are wrapped in try-catch blocks with error logging
- ConfigLoader uses Zod schema validation for all configuration objects before accepting values
- Parse failures generate console.warn log entries with file path and error message context
- Module-specific prefixes are included in all configuration parse failure log messages
- Non-required configuration fields use Zod's .optional() modifier
- Configuration parsing failures return null or default objects rather than throwing exceptions

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse and TOML.parse operations in configuration modules must be wrapped in try-catch blocks with appropriate logging and Zod schema validation before accepting parsed values.
</enforcement>
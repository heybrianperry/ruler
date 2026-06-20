# Use console.warn for Configuration Loading Errors in Secure Coding Contexts: Environment Variable Access

These rules are ALWAYS ACTIVE for all agent and configuration loading code that handles file system operations, configuration parsing, and environment variable access for configuration purposes.

### Rules

- **R-CONFIG-001** SHOULD: Environment variable access for configuration (process.env) SHOULD be treated as external input requiring validation.
- **R-CONFIG-002** MUST: All configuration loading code MUST use console.warn for non-fatal errors with file path and error context.
- **R-CONFIG-003** MUST: All external configuration input MUST be validated using JSON.parse or zod schemas before use.
- **R-CONFIG-004** MUST: No unvalidated configuration data from process.env or file system MUST be used directly in application logic.
- **R-CONFIG-005** SHOULD: Configuration file path and error message SHOULD be included in all warning logs to aid debugging.
- **R-CONFIG-006** SHOULD: Each configuration parameter SHOULD be evaluated to determine if it is optional (warn and continue) or required (throw exception).

### Verify

```bash
# Count console.warn usage in configuration-related code
grep -r "console\.warn" src/ | grep -E "(config|Config)" | wc -l

# Identify unvalidated JSON.parse calls
grep -r "JSON\.parse" src/ | grep -v "try" | wc -l

# Verify zod schema usage in ConfigLoader
grep -r "z\.object\|z\.string\|z\.boolean" src/core/ConfigLoader.ts | wc -l
```

**Accept when:**
- All configuration loading code uses console.warn for non-fatal errors with file path and error context
- All external configuration input is validated using JSON.parse or zod schemas before use
- No unvalidated configuration data from process.env or file system is used directly in application logic
- Configuration loading modules (ConfigLoader.ts, FirebenderAgent.ts) follow the established pattern
- File system operations that read configuration files include appropriate error handling

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Code review of configuration loading modules and static analysis to detect unvalidated external input usage are mandatory before merging configuration loading code.
</enforcement>
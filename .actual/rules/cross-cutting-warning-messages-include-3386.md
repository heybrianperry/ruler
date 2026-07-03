# Use console.warn for Configuration Loading Errors in Secure Coding Contexts: Warning Messages Include

These rules are ALWAYS ACTIVE for all configuration loading code that handles file system operations and configuration parsing, including ConfigLoader.ts, FirebenderAgent.ts, and related modules that read configuration files (JSON, TOML) and access environment variables for configuration purposes.

### Rules

- **R-CONFIG-001** MUST: Warning messages MUST include sufficient context to identify the configuration source (file path, environment variable) and the nature of the failure.

### Verify

```bash
# Count console.warn usage in configuration-related code
grep -r "console\.warn" src/ | grep -E "(config|Config)" | wc -l

# Identify unvalidated JSON.parse calls (should be wrapped in try-catch)
grep -r "JSON\.parse" src/ | grep -v "try" | wc -l

# Verify zod schema usage in ConfigLoader
grep -r "z\.object\|z\.string\|z\.boolean" src/core/ConfigLoader.ts | wc -l
```

**Accept when:**
- All configuration loading code uses console.warn for non-fatal errors with file path and error context
- All external configuration input is validated using JSON.parse or zod schemas before use
- No unvalidated configuration data from process.env or file system is used directly in application logic
- Warning messages include the configuration file path and descriptive error message to aid debugging

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Configuration loading errors must be logged with sufficient context, and all external configuration input must be validated before use.
</enforcement>
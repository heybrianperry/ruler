# Use console.warn for Configuration Loading Errors in Secure Coding Contexts: Configuration Loading Code

These rules are ALWAYS ACTIVE for all agent and configuration loading code that handles file system operations and configuration parsing, including ConfigLoader.ts, FirebenderAgent.ts, and related modules that read configuration files (JSON, TOML) or access environment variables for configuration purposes.

### Rules

- **R-CONFIG-001** MUST: Configuration loading code MUST use console.warn to log file read failures and parsing errors with descriptive context including file paths and error messages.
- **R-CONFIG-002** MUST: All external configuration input MUST be validated using JSON.parse or zod schemas before use in application logic.
- **R-CONFIG-003** MUST: No unvalidated configuration data from process.env or file system operations MUST be used directly in application logic.
- **R-CONFIG-004** SHOULD: Include the configuration file path and error message in all warning logs to aid debugging.
- **R-CONFIG-005** SHOULD: Document whether each configuration parameter is optional (warn and continue) or required (throw exception).

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
- Configuration file paths and error messages are included in warning logs
- Configuration parameters are documented as optional or required

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules during code review of configuration loading modules. Static analysis and grep-based verification MUST be performed before accepting configuration loading code changes.
</enforcement>
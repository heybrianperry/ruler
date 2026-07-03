# Use console.warn for Configuration Loading Errors in Secure Coding Contexts: External Configuration Input

These rules are ALWAYS ACTIVE for all agent and configuration loading code that handles file system operations and configuration parsing, including ConfigLoader.ts, FirebenderAgent.ts, and any code that reads configuration files (JSON, TOML) or accesses environment variables for configuration purposes.

### Rules

- **R-EXTCONF-001** MUST: All external configuration input MUST be validated using explicit parsing mechanisms (JSON.parse, zod schemas) before use.
- **R-EXTCONF-002** MUST: All configuration loading code MUST use console.warn for non-fatal errors with file path and error context included.
- **R-EXTCONF-003** MUST: No unvalidated configuration data from process.env or file system operations MUST be used directly in application logic.
- **R-EXTCONF-004** SHOULD: Configuration parameters SHOULD be documented as either optional (warn and continue) or required (throw exception).
- **R-EXTCONF-005** SHOULD: Warning message content SHOULD be reviewed to ensure no secrets or sensitive data are logged; paths SHOULD be sanitized if necessary.

### Verify

```bash
# Count console.warn usage in configuration-related code
grep -r "console\.warn" src/ | grep -E "(config|Config)" | wc -l

# Detect unvalidated JSON.parse calls (should be wrapped in try-catch)
grep -r "JSON\.parse" src/ | grep -v "try" | wc -l

# Verify zod schema usage in ConfigLoader
grep -r "z\.object\|z\.string\|z\.boolean" src/core/ConfigLoader.ts | wc -l
```

**Accept when:**
- All configuration loading code uses console.warn for non-fatal errors with file path and error context
- All external configuration input is validated using JSON.parse or zod schemas before use
- No unvalidated configuration data from process.env or file system is used directly in application logic
- Configuration parameters are documented as optional or required
- Warning messages do not expose secrets or sensitive file paths

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules when reviewing configuration loading code. All violations MUST be flagged during code review with specific guidance to add console.warn logging and input validation before merging.
</enforcement>
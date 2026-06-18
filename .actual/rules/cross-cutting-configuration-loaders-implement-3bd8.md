# Use console.warn for Configuration Loading Errors in Secure Coding Contexts: Configuration Loaders Implement

These rules are ALWAYS ACTIVE for all agent and configuration loading code that handles file system operations and configuration parsing, including ConfigLoader.ts, FirebenderAgent.ts, and any modules that read configuration files (JSON, TOML) or access environment variables for configuration purposes.

### Rules

- **R-CONFIG-001** SHOULD: Configuration loaders SHOULD implement fallback behavior that allows execution to continue when non-critical configuration is unavailable.
- **R-CONFIG-002** MUST: All configuration loading code MUST use console.warn for non-fatal errors with file path and error context included.
- **R-CONFIG-003** MUST: All external configuration input MUST be validated using JSON.parse or zod schemas before use in application logic.
- **R-CONFIG-004** MUST: No unvalidated configuration data from process.env or file system operations MUST be used directly in application logic.
- **R-CONFIG-005** SHOULD: Configuration parameters SHOULD be documented as either optional (warn and continue) or required (throw exception).
- **R-CONFIG-006** SHOULD: Warning messages SHOULD be reviewed to ensure no secrets or sensitive data are logged; paths SHOULD be sanitized if necessary.

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
- Configuration parameters are documented as optional or required
- Warning messages do not expose sensitive information or secrets

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All configuration loading code MUST comply with R-CONFIG-001 through R-CONFIG-006 before merge. Violations require code review feedback and remediation or documented exception approval from the security team.
</enforcement>
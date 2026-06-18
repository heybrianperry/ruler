# Use console.warn for Configuration Loading Errors in Secure Coding Contexts: Configuration Loading Use

These rules are ALWAYS ACTIVE for all agent and configuration loading code that handles file system operations and configuration parsing, including ConfigLoader.ts, FirebenderAgent.ts, and related modules that read configuration files (JSON, TOML) and access environment variables for configuration purposes.

### Rules

- **R-CONFIG-001** MAY: Configuration loading MAY use structured schema validation libraries (such as zod) to enforce type safety and validation rules on external configuration data before use in application logic.

### Verify

```bash
# Count console.warn usage in configuration-related code
grep -r "console\.warn" src/ | grep -E "(config|Config)" | wc -l

# Identify unvalidated JSON.parse calls (should be wrapped in try-catch)
grep -r "JSON\.parse" src/ | grep -v "try" | wc -l

# Verify zod schema validation is present in ConfigLoader.ts
grep -r "z\.object\|z\.string\|z\.boolean" src/core/ConfigLoader.ts | wc -l
```

**Accept when:**
- All configuration loading code uses console.warn for non-fatal errors with file path and error context
- All external configuration input is validated using JSON.parse or zod schemas before use
- No unvalidated configuration data from process.env or file system is used directly in application logic
- Configuration file paths and error messages are included in all warning logs to aid debugging
- Optional vs required configuration parameters are clearly distinguished in implementation

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration loading code MUST be reviewed to ensure console.warn is used for non-fatal errors, external input is validated via JSON.parse or zod schemas, and no unvalidated configuration data flows into application logic. Violations require code review feedback and remediation before merge.
</enforcement>
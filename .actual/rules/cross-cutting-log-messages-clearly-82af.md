# Use console.warn for Configuration Error Logging in Agent Modules: Log Messages Clearly

These rules are ALWAYS ACTIVE for all agent modules that handle configuration loading and parsing operations, particularly those using Node.js fs and path modules for file system operations and JSON.parse for configuration file content.

### Rules

- **R-CONFIG-001** SHOULD: Log messages SHOULD clearly identify the configuration file path or name that failed to load or parse.

### Verify

```bash
# Check for console.warn usage in configuration error contexts
grep -r "console\.warn.*Failed to read" src/agents/

# Verify JSON.parse error handling patterns
grep -r "JSON\.parse" src/agents/ | grep -A 5 "catch"

# Run configuration error tests
npm test -- --grep "configuration.*error" 2>&1 | grep -i warn
```

**Accept when:**
- All agent modules that load configuration files use console.warn for non-fatal read/parse errors
- Error messages include file path and error details using template literals (e.g., `console.warn(\`Failed to read/parse existing firebender.json: ${error}\`)`)
- Configuration loading functions handle exceptions without propagating them when graceful degradation is implemented
- JSON.parse calls are wrapped in try-catch blocks within configuration loading functions
- Configuration loading functions return default configuration objects or null values after logging warnings

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules during code review and CI pipeline checks. All agent modules loading configuration from JSON files MUST comply with R-CONFIG-001 unless an explicit exception is documented and approved by the module owner.
</enforcement>
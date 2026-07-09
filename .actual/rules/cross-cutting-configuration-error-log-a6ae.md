# Use console.warn for Configuration Error Logging in Agent Modules: Configuration Error Log

These rules are ALWAYS ACTIVE for all agent modules that handle configuration loading and parsing operations, particularly those that load configuration from JSON files at initialization time.

### Rules

- **R-CONFIG-001** MUST: Configuration error log messages MUST include the error details using template literals to capture the error object or message.

### Verify

```bash
# Check for console.warn usage with template literals in configuration error contexts
grep -r "console\.warn.*Failed to read" src/agents/

# Verify JSON.parse calls are wrapped in try-catch blocks
grep -r "JSON\.parse" src/agents/ | grep -A 5 "catch"

# Run tests for configuration error logging
npm test -- --grep "configuration.*error" 2>&1 | grep -i warn
```

**Accept when:**
- All agent modules that load configuration files use console.warn for non-fatal read/parse errors
- Error messages include file path and error details using template literals (e.g., `console.warn(\`Failed to read/parse existing firebender.json: ${error}\`)`)
- Configuration loading functions handle exceptions without propagating them when graceful degradation is implemented
- JSON.parse operations are wrapped in try-catch blocks within configuration loading functions

<enforcement>
Claude Code MUST NOT skip or defer verification of R-CONFIG-001. All agent modules loading configuration must use console.warn with template literal error interpolation for non-fatal configuration errors.
</enforcement>
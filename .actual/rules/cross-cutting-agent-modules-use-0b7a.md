# Use console.warn for Configuration Error Logging in Agent Modules: Agent Modules Use

These rules are ALWAYS ACTIVE for all agent modules that handle configuration loading and parsing operations.

### Rules

- **R-AGENT-CONFIG-001** MUST: Agent modules MUST use console.warn to log configuration file read or parse errors that do not prevent agent initialization.

### Verify

```bash
# Check for console.warn usage in configuration error handling
grep -r "console\.warn.*Failed to read" src/agents/

# Verify JSON.parse is wrapped in try-catch blocks
grep -r "JSON\.parse" src/agents/ | grep -A 5 "catch"

# Run tests for configuration error scenarios
npm test -- --grep "configuration.*error" 2>&1 | grep -i warn
```

**Accept when:**
- All agent modules that load configuration files use console.warn for non-fatal read/parse errors
- Error messages include file path and error details using template literals
- Configuration loading functions handle exceptions without propagating them when graceful degradation is implemented
- JSON.parse operations are wrapped in try-catch blocks within configuration loading functions
- Agent initialization succeeds even when configuration files are missing or malformed

<enforcement>
Claude Code MUST NOT skip or defer verification of console.warn usage in agent module configuration error handling. All agent modules loading configuration from JSON files MUST comply with R-AGENT-CONFIG-001.
</enforcement>
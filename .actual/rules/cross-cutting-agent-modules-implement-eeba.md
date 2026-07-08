# Use console.warn for Configuration Error Logging in Agent Modules: Agent Modules Implement

These rules are ALWAYS ACTIVE for all agent modules that handle configuration loading and parsing operations.

### Rules

- **R-AGENT-CONFIG-001** MAY: Agent modules MAY implement fallback configuration behavior after logging configuration errors with console.warn

### Verify

```bash
# Check for console.warn usage in configuration error handling
grep -r "console\.warn.*Failed to read" src/agents/

# Verify JSON.parse is wrapped in try-catch blocks
grep -r "JSON\.parse" src/agents/ | grep -A 5 "catch"

# Run configuration error tests
npm test -- --grep "configuration.*error" 2>&1 | grep -i warn
```

**Accept when:**
- All agent modules that load configuration files use console.warn for non-fatal read/parse errors
- Error messages include file path and error details using template literals
- Configuration loading functions handle exceptions without propagating them when graceful degradation is implemented
- Agent modules can initialize successfully with missing or malformed configuration files

<enforcement>
Claude Code MUST NOT skip or defer verification of console.warn usage in agent module configuration loading paths.
</enforcement>
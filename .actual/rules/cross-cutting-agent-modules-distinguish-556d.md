# Use console.warn for Configuration Error Logging in Agent Modules: Agent Modules Distinguish

These rules are ALWAYS ACTIVE for all agent modules that handle configuration loading and parsing operations.

### Rules

- **R-AGENT-CONFIG-001** SHOULD: Agent modules SHOULD distinguish between missing configuration files and malformed configuration content in log messages.

### Verify

```bash
# Check for console.warn usage in agent configuration loading
grep -r "console\.warn.*Failed to read" src/agents/

# Verify JSON.parse error handling patterns
grep -r "JSON\.parse" src/agents/ | grep -A 5 "catch"

# Run configuration error tests
npm test -- --grep "configuration.*error" 2>&1 | grep -i warn
```

**Accept when:**
- All agent modules that load configuration files use console.warn for non-fatal read/parse errors
- Error messages include file path and error details using template literals
- Configuration loading functions handle exceptions without propagating them when graceful degradation is implemented
- Error messages distinguish between file-not-found scenarios and JSON parsing failures

<enforcement>
Claude Code MUST NOT skip or defer verification of console.warn usage patterns in agent configuration loading. All agent modules handling configuration must be checked against these rules during code review.
</enforcement>
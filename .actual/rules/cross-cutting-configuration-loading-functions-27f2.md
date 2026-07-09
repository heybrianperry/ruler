# Use console.warn for Configuration Error Logging in Agent Modules: Configuration Loading Functions

These rules are ALWAYS ACTIVE for all agent modules that handle configuration loading and parsing operations, particularly those using Node.js fs and path modules for file system operations and JSON.parse for configuration file content.

### Rules

- **R-CONFIG-001** MUST: Configuration loading functions MUST handle JSON.parse exceptions and log them without propagating to callers when graceful degradation is acceptable.
- **R-CONFIG-002** MUST: Use console.warn with template literals for non-fatal configuration errors, including file path and error details (e.g., `console.warn(\`Failed to read/parse existing config.json: ${error}\`)`).
- **R-CONFIG-003** MUST: Wrap JSON.parse calls in try-catch blocks within configuration loading functions to prevent unhandled exceptions.
- **R-CONFIG-004** SHOULD: Return default configuration objects or null values after logging warnings to enable graceful degradation when configuration files are missing or corrupted.
- **R-CONFIG-005** SHOULD: Document expected configuration file locations and formats in module README to help operators diagnose warning messages.

### Verify

```bash
# Check for console.warn usage in configuration loading functions
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
- JSON.parse operations are wrapped in try-catch blocks
- Agents can initialize successfully with missing or malformed configuration files

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration loading functions in agent modules MUST comply with R-CONFIG-001 through R-CONFIG-005 before code review approval.
</enforcement>
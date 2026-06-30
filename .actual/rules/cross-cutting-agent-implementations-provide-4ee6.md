# Validate JSON Input from External Configuration Files in Agent Implementations: Agent Implementations Provide

These rules are ALWAYS ACTIVE for all agent implementations that consume external JSON configuration files from the filesystem.

### Rules

- **R-AGENT-CONFIG-001** MAY: Agent implementations MAY provide default fallback configurations when optional configuration files are missing.

### Verify

```bash
# Detect unvalidated JSON.parse operations in agent implementations
grep -r 'JSON\.parse' src/agents/ | grep -v 'try\|catch' # Should return no matches after implementation

# Find direct readFile + JSON.parse patterns that need validation wrapping
grep -r 'readFile.*JSON\.parse' src/agents/ # Should be replaced with validated configuration loading

# Run configuration validation test suite
npm test -- --grep 'configuration validation' # Should pass all validation test cases
```

**Accept when:**
- All JSON.parse operations on external file content in agent implementations are wrapped in try-catch blocks with appropriate error handling
- Schema validation is applied to all configuration objects before they are consumed by agent implementations
- Unit tests demonstrate that malformed JSON and invalid configuration schemas are rejected with descriptive error messages
- Default fallback configurations are provided for optional configuration files that are missing

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis tools (ESLint rules) MUST detect unvalidated JSON.parse usage. Code review MUST verify validation for all external configuration loading. Integration tests MUST verify validation behavior in CI pipeline. CI pipeline MUST fail if static analysis detects unvalidated JSON.parse on external file content.
</enforcement>
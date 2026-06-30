# Validate JSON Input from External Configuration Files in Agent Implementations: Agent Implementations Not

These rules are ALWAYS ACTIVE for all agent implementations that consume external configuration files via filesystem operations.

### Rules

- **R-AGENT-CONFIG-001** MUST_NOT: Agent implementations MUST NOT proceed with initialization if configuration validation fails.

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
- All JSON.parse operations on external file content are wrapped in try-catch blocks with appropriate error handling
- Schema validation is applied to all configuration objects before they are consumed by agent implementations
- Unit tests demonstrate that malformed JSON and invalid configuration schemas are rejected with descriptive error messages
- Configuration files loaded via fs, fs/promises, or FileSystemUtils modules are validated before use
- MCP configuration files (mcpPath) and similar external configuration sources are validated before parsing

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis tools (ESLint rules) MUST detect unvalidated JSON.parse usage. Code review MUST block merges lacking validation for external configuration loading. CI pipeline MUST fail if unvalidated JSON.parse on external file content is detected.
</enforcement>
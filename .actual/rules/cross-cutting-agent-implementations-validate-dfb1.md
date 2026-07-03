# Validate JSON Input from External Configuration Files in Agent Implementations: Agent Implementations Validate

These rules are ALWAYS ACTIVE for all agent implementations that consume external configuration files via filesystem operations.

### Rules

- **R-AGENT-VALIDATE-001** MUST: All agent implementations MUST validate JSON configuration file content using a schema validation library before consuming parsed data.

### Verify

```bash
# Detect unvalidated JSON.parse operations in agent implementations
grep -r 'JSON\.parse' src/agents/ | grep -v 'try\|catch'

# Find readFile operations followed by JSON.parse without validation
grep -r 'readFile.*JSON\.parse' src/agents/

# Run configuration validation test suite
npm test -- --grep 'configuration validation'
```

**Accept when:**
- All JSON.parse operations on external file content are wrapped in try-catch blocks with appropriate error handling
- Schema validation is applied to all configuration objects before they are consumed by agent implementations
- Unit tests demonstrate that malformed JSON and invalid configuration schemas are rejected with descriptive error messages
- Configuration files loaded via fs, fs/promises, or FileSystemUtils modules are validated against a schema before use
- MCP configuration files (mcpPath) and similar external configuration sources are validated before parsing

<enforcement>
Claude Code MUST NOT skip or defer verification of this rule. Static analysis tools (ESLint rules) MUST detect unvalidated JSON.parse usage. Code review MUST block merges if validation is missing from agent implementations. CI pipeline MUST fail if unvalidated JSON.parse on external file content is detected.
</enforcement>
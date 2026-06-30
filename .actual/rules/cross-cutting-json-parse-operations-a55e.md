# Validate JSON Input from External Configuration Files in Agent Implementations: Json Parse Operations

These rules are ALWAYS ACTIVE for all agent implementations that consume external JSON configuration files from the filesystem.

### Rules

- **R-JSON-001** MUST: JSON.parse operations on external file content MUST be wrapped in try-catch blocks to handle malformed JSON gracefully.

### Verify

```bash
# Verify no unvalidated JSON.parse operations exist in agent implementations
grep -r 'JSON\.parse' src/agents/ | grep -v 'try\|catch' # Should return no matches after implementation

# Verify readFile operations are using validated configuration loading
grep -r 'readFile.*JSON\.parse' src/agents/ # Should be replaced with validated configuration loading

# Verify configuration validation tests pass
npm test -- --grep 'configuration validation' # Should pass all validation test cases
```

**Accept when:**
- All JSON.parse operations on external file content are wrapped in try-catch blocks with appropriate error handling
- Schema validation is applied to all configuration objects before they are consumed by agent implementations
- Unit tests demonstrate that malformed JSON and invalid configuration schemas are rejected with descriptive error messages
- Static analysis tools confirm no unvalidated JSON.parse usage on external file content remains

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse operations on external configuration files must be validated before merging. CI pipeline MUST fail if static analysis detects unvalidated JSON.parse on external file content.
</enforcement>
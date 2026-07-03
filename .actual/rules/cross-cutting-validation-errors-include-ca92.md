# Validate JSON Input from External Configuration Files in Agent Implementations: Validation Errors Include

These rules are ALWAYS ACTIVE for all agent implementations that consume external JSON configuration files from the filesystem.

### Rules

- **R-VALIDATE-001** SHOULD: Validation errors SHOULD include descriptive messages indicating which configuration file failed validation and why.

### Verify

```bash
# Detect unvalidated JSON.parse operations on external file content
grep -r 'JSON\.parse' src/agents/ | grep -v 'try\|catch' # Should return no matches after implementation

# Find readFile operations followed by JSON.parse that should be replaced
grep -r 'readFile.*JSON\.parse' src/agents/ # Should be replaced with validated configuration loading

# Run configuration validation test suite
npm test -- --grep 'configuration validation' # Should pass all validation test cases
```

**Accept when:**
- All JSON.parse operations on external file content are wrapped in try-catch blocks with appropriate error handling
- Schema validation is applied to all configuration objects before they are consumed by agent implementations
- Unit tests demonstrate that malformed JSON and invalid configuration schemas are rejected with descriptive error messages
- Validation error messages include the configuration file path and reason for validation failure

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis tools (ESLint rules) MUST detect unvalidated JSON.parse usage. Code review MUST verify validation is present for all external configuration loading. Integration tests MUST verify validation behavior in CI pipeline.
</enforcement>
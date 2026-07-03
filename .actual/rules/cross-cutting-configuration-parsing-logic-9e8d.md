# Validate JSON Input from External Configuration Files in Agent Implementations: Configuration Parsing Logic

These rules are ALWAYS ACTIVE for all agent implementations that consume external configuration files, particularly those extending the IAgent interface and loading JSON configuration via fs, fs/promises, or FileSystemUtils modules.

### Rules

- **R-CONFIG-001** MUST: All JSON.parse operations on external file content MUST be wrapped in try-catch blocks with appropriate error handling.
- **R-CONFIG-002** MUST: Schema validation MUST be applied to all configuration objects before they are consumed by agent implementations.
- **R-CONFIG-003** SHOULD: Configuration parsing logic SHOULD be centralized in a shared utility module to ensure consistent validation across all agent implementations.
- **R-CONFIG-004** MUST: Malformed JSON and invalid configuration schemas MUST be rejected with descriptive error messages.
- **R-CONFIG-005** MUST: Configuration files loaded via fs, fs/promises, or FileSystemUtils modules MUST undergo validation before use in agent initialization.

### Verify

```bash
# Detect unvalidated JSON.parse operations in agent implementations
grep -r 'JSON\.parse' src/agents/ | grep -v 'try\|catch'

# Find direct readFile + JSON.parse patterns that should be replaced
grep -r 'readFile.*JSON\.parse' src/agents/

# Run configuration validation test suite
npm test -- --grep 'configuration validation'
```

**Accept when:**
- All JSON.parse operations on external file content are wrapped in try-catch blocks with appropriate error handling
- Schema validation is applied to all configuration objects before they are consumed by agent implementations
- Unit tests demonstrate that malformed JSON and invalid configuration schemas are rejected with descriptive error messages
- A shared ConfigurationLoader utility exists that encapsulates file reading, parsing, and validation
- Configuration schemas are defined as TypeScript types with corresponding runtime validators

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis tools (ESLint rules) MUST detect unvalidated JSON.parse usage. Code review MUST require validation for all external configuration loading. CI pipeline MUST fail if unvalidated JSON.parse on external file content is detected.
</enforcement>
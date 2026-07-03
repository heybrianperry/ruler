# Validate Untrusted Input from Configuration Files Before Parsing: Parsing Operations Wrapped

These rules are ALWAYS ACTIVE for all code that reads and parses external configuration files, including VSCode settings.json, OpenHands TOML, and any other untrusted configuration sources loaded from disk.

### Rules

- **R-PARSE-001** MUST: Wrap all parsing operations (JSON.parse(), parseTOML(), or similar deserialization functions) in try-catch blocks to handle malformed input gracefully without crashing the application.
- **R-PARSE-002** MUST: Apply validation to all functions that read configuration files using fs or fs/promises modules before storing parsed data in cache layers or exposing through public APIs.
- **R-PARSE-003** MUST: Validate parsed configuration data against a schema (using zod, joi, or equivalent) before storing in Map-based cache layers (existingServerMap, existingSseServers, existingStdioServers) or passing to downstream consumers.
- **R-PARSE-004** MUST: Return meaningful error messages when configuration validation fails, enabling users to identify and correct invalid configuration files.
- **R-PARSE-005** SHOULD: Define and document configuration schemas for all external configuration file formats to establish clear validation requirements.

### Verify

```bash
# Check for unvalidated JSON.parse calls
grep -r 'JSON\.parse' src/ | grep -v 'validate\|schema\|parse.*catch' || echo 'All JSON.parse calls are validated'

# Check for unvalidated parseTOML calls
grep -r 'parseTOML' src/ | grep -v 'validate\|schema\|parse.*catch' || echo 'All parseTOML calls are validated'

# Run configuration validation tests
npm test -- --grep 'configuration.*validation' && echo 'Configuration validation tests pass'
```

**Accept when:**
- All grep commands return no unvalidated parsing calls or echo success messages
- Configuration validation test suite passes with 100% coverage of validation logic
- Manual testing confirms that malformed configuration files are rejected with clear error messages without crashing the application
- All public API functions that expose configuration data (readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands) include validation before returning data

<enforcement>
Claude Code MUST NOT skip or defer verification. All parsing operations must be wrapped in try-catch blocks and validated against schemas before use. Violations block CI builds and require security team review.
</enforcement>
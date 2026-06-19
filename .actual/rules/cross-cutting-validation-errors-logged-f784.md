# Validate Untrusted Input from Configuration Files Before Parsing: Validation Errors Logged

These rules are ALWAYS ACTIVE for all code that reads and parses external configuration files, including VSCode settings.json, OpenHands TOML, and any other untrusted configuration sources loaded from disk.

### Rules

- **R-CONFIG-001** MUST: Validate all configuration file input using a schema validation library (zod, joi, or equivalent) before storing in cache layers or exposing through public APIs.
- **R-CONFIG-002** MUST: Wrap all JSON.parse() and parseTOML() calls with validation functions that return Result<T, ValidationError> types.
- **R-CONFIG-003** MAY: Log validation errors with sanitized error messages that do not expose sensitive file paths or content.
- **R-CONFIG-004** MUST: Reject malformed configuration files with clear, actionable error messages without crashing the application.
- **R-CONFIG-005** MUST: Define and maintain schemas for all configuration structures (VSCodeSettings, OpenHands MCP configuration) that evolve with format changes.
- **R-CONFIG-006** MUST: Add unit tests verifying rejection of malformed configuration files with various invalid inputs (missing fields, wrong types, malicious content).

### Verify

```bash
# Detect unvalidated JSON.parse calls in configuration loading paths
grep -r 'JSON\.parse' src/ | grep -v 'validate\|schema\|parse.*catch' || echo 'All JSON.parse calls are validated'

# Detect unvalidated parseTOML calls
grep -r 'parseTOML' src/ | grep -v 'validate\|schema\|parse.*catch' || echo 'All parseTOML calls are validated'

# Run configuration validation test suite
npm test -- --grep 'configuration.*validation' && echo 'Configuration validation tests pass'
```

**Accept when:**
- All grep commands return no unvalidated parsing calls or echo success messages
- Configuration validation test suite passes with 100% coverage of validation logic
- Manual testing confirms that malformed configuration files are rejected with clear error messages without crashing the application
- Schema definitions exist for all configuration structures and are documented in user-facing documentation

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis in CI pipeline MUST detect unvalidated JSON.parse() and parseTOML() calls. Code review MUST enforce validation for all new configuration parsing code. Pull requests MUST be blocked until validation is added and tests demonstrate rejection of invalid input.
</enforcement>
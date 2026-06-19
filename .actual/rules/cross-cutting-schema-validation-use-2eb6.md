# Validate Untrusted Input from Configuration Files Before Parsing: Schema Validation Use

These rules are ALWAYS ACTIVE for all code that reads and parses external configuration files, including VSCode settings.json, OpenHands TOML, and any other untrusted configuration sources loaded from disk.

### Rules

- **R-CONFIG-001** SHOULD: Schema validation SHOULD use a validation library (e.g., zod, joi, ajv) rather than manual field checking.

### Verify

```bash
# Verify all JSON.parse calls are validated
grep -r 'JSON\.parse' src/ | grep -v 'validate\|schema\|parse.*catch' || echo 'All JSON.parse calls are validated'

# Verify all parseTOML calls are validated
grep -r 'parseTOML' src/ | grep -v 'validate\|schema\|parse.*catch' || echo 'All parseTOML calls are validated'

# Verify configuration validation tests pass
npm test -- --grep 'configuration.*validation' && echo 'Configuration validation tests pass'
```

**Accept when:**
- All grep commands return no unvalidated parsing calls or echo success messages
- Configuration validation test suite passes with 100% coverage of validation logic
- Manual testing confirms that malformed configuration files are rejected with clear error messages without crashing the application

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration parsing operations must be wrapped in schema validation before merging.
</enforcement>
# Validate Untrusted Input from Configuration Files Before Parsing: Parsed Configuration Objects

These rules are ALWAYS ACTIVE for all code that reads and parses external configuration files, including VSCode settings.json, OpenHands TOML, and any other untrusted configuration sources stored on disk.

### Rules

- **R-CFG-001** MUST: Parsed configuration objects MUST be validated for required fields and type correctness before insertion into cache layers (Map structures).
- **R-CFG-002** MUST: All functions that read configuration files using fs or fs/promises modules MUST wrap parsing operations (JSON.parse(), parseTOML(), or similar deserialization) with schema validation.
- **R-CFG-003** MUST: Public API functions that expose configuration data (readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands) MUST validate parsed configuration before returning to callers.
- **R-CFG-004** SHOULD: Use TypeScript type guards with runtime type checking libraries (zod, io-ts) for validation rather than manual field-by-field validation.
- **R-CFG-005** SHOULD: Validation functions SHOULD return Result<T, ValidationError> types to enable clear error handling and propagation.

### Verify

```bash
# Detect unvalidated JSON.parse calls in configuration parsing code
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
- Schema definitions exist for VSCodeSettings and OpenHands TOML configuration structures
- Unit tests verify rejection of malformed configuration files with various invalid inputs (missing fields, wrong types, malicious content)

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis in CI pipeline MUST detect unvalidated JSON.parse() and parseTOML() calls. Code review MUST enforce validation for all new configuration parsing code. Pull requests MUST be blocked until validation is added and tests demonstrate rejection of invalid input.
</enforcement>
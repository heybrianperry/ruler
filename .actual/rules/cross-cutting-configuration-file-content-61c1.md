# Validate Untrusted Input from Configuration Files Before Parsing: Configuration File Content

These rules are ALWAYS ACTIVE for all code that reads and parses external configuration files from disk, including VSCode settings.json, OpenHands TOML files, and any other untrusted configuration sources.

### Rules

- **R-CFG-001** MUST: All configuration file content read from disk MUST be validated against a defined schema before parsing with JSON.parse() or parseTOML().
- **R-CFG-002** MUST: All functions that read configuration files using fs or fs/promises modules MUST implement schema validation before deserialization.
- **R-CFG-003** MUST: All parsing operations using JSON.parse(), parseTOML(), or similar deserialization functions MUST be wrapped in validation functions that return Result<T, ValidationError> types.
- **R-CFG-004** MUST: All code paths that store parsed configuration in Map-based cache layers (existingServerMap, existingSseServers, existingStdioServers) MUST validate input before cache insertion.
- **R-CFG-005** MUST: Public API functions that expose configuration data (readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands) MUST handle validation failures and return meaningful error messages.
- **R-CFG-006** SHOULD: Use TypeScript type guards with runtime type checking libraries (zod, io-ts) for validation to provide both compile-time type safety and runtime validation.
- **R-CFG-007** SHOULD: Configuration validation schemas SHOULD be permissive for optional fields to avoid rejecting valid edge cases.
- **R-CFG-008** SHOULD: Validation error messages SHOULD be clear and actionable for users to understand configuration requirements.

### Verify

```bash
# Verify no unvalidated JSON.parse calls exist
grep -r 'JSON\.parse' src/ | grep -v 'validate\|schema\|parse.*catch' || echo 'All JSON.parse calls are validated'

# Verify no unvalidated parseTOML calls exist
grep -r 'parseTOML' src/ | grep -v 'validate\|schema\|parse.*catch' || echo 'All parseTOML calls are validated'

# Verify configuration validation tests pass
npm test -- --grep 'configuration.*validation' && echo 'Configuration validation tests pass'
```

**Accept when:**
- All grep commands return no unvalidated parsing calls or echo success messages
- Configuration validation test suite passes with 100% coverage of validation logic
- Manual testing confirms that malformed configuration files are rejected with clear error messages without crashing the application
- Schema definitions exist for VSCodeSettings and OpenHands TOML configuration structures
- Unit tests verify rejection of malformed configuration files with various invalid inputs (missing fields, wrong types, malicious content)

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis in CI pipeline MUST detect unvalidated JSON.parse() and parseTOML() calls. Code review MUST require validation for all new configuration parsing code. Pull requests MUST be blocked until validation is added and tests demonstrate rejection of invalid input.
</enforcement>
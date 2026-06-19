# Validate Untrusted Input from Configuration Files Before Parsing: Configuration Data Not

These rules are ALWAYS ACTIVE for all code that reads and parses external configuration files, including VSCode settings.json, OpenHands TOML, and any other untrusted configuration sources loaded from disk.

### Rules

- **R-CONFIG-001** MUST NOT: Configuration data MUST NOT be passed to downstream APIs or stored in cache layers without successful validation against a defined schema.
- **R-CONFIG-002** MUST: All JSON.parse() and parseTOML() calls that process external configuration files MUST be wrapped in validation functions that return Result<T, ValidationError> types.
- **R-CONFIG-003** MUST: Configuration parsing functions (readVSCodeSettings, propagateMcpToOpenHands, and similar public APIs) MUST validate input before storing in Map-based cache layers (existingServerMap, existingSseServers, existingStdioServers).
- **R-CONFIG-004** MUST: Schema validation MUST use a runtime validation library (zod, joi, or equivalent) that provides both compile-time type safety and runtime validation.
- **R-CONFIG-005** MUST: Validation failures MUST return meaningful error messages that distinguish between malformed input and logic errors, enabling fail-fast behavior.
- **R-CONFIG-006** SHOULD: Configuration schemas SHOULD be permissive for optional fields to avoid rejecting valid edge cases and breaking backward compatibility.
- **R-CONFIG-007** SHOULD: Validated configuration results SHOULD be cached to avoid repeated validation overhead on large configuration files.

### Verify

```bash
# Detect unvalidated JSON.parse calls in configuration parsing code
grep -r 'JSON\.parse' src/ | grep -v 'validate\|schema\|parse.*catch' || echo 'All JSON.parse calls are validated'

# Detect unvalidated parseTOML calls
grep -r 'parseTOML' src/ | grep -v 'validate\|schema\|parse.*catch' || echo 'All parseTOML calls are validated'

# Verify configuration validation test suite passes
npm test -- --grep 'configuration.*validation' && echo 'Configuration validation tests pass'
```

**Accept when:**
- All grep commands return no unvalidated parsing calls or echo success messages
- Configuration validation test suite passes with 100% coverage of validation logic
- Manual testing confirms that malformed configuration files are rejected with clear error messages without crashing the application
- Schema definitions exist for VSCodeSettings and OpenHands TOML configuration structures
- Unit tests verify rejection of malformed configuration files with various invalid inputs (missing fields, wrong types, malicious content)

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration parsing code paths MUST be validated before merging. Static analysis in CI MUST detect unvalidated JSON.parse() and parseTOML() calls and fail the build.
</enforcement>
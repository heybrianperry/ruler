# Enforce JSON.parse and TOML Parsing with Try-Catch for Configuration and External Data: Agent Implementations Use

These rules are ALWAYS ACTIVE for all agent implementations, configuration loaders, MCP path handlers, VSCode settings managers, and any module that parses external data from JSON or TOML sources originating from file system, environment variables, or external sources.

### Rules

- **R-JSON-TOML-001** SHOULD: Agent implementations SHOULD use console.warn or equivalent logging for non-fatal parse failures and console.error for fatal parse failures that prevent agent initialization.
- **R-JSON-TOML-002** MUST: All JSON.parse and parseTOML calls on external data sources MUST be wrapped in try-catch blocks or documented error boundaries, except where explicitly excepted (EXC-001 or EXC-002).
- **R-JSON-TOML-003** SHOULD: Parse error handlers SHOULD log appropriate context including file path and operation type, but SHOULD avoid logging raw malformed content that might contain sensitive data or be excessively large.
- **R-JSON-TOML-004** SHOULD: Configuration loaders SHOULD propagate parse errors to the caller with context about which file failed, allowing the application to decide whether to fail-fast or fall back to defaults.
- **R-JSON-TOML-005** MAY: Developers MAY implement a centralized parseJSON utility function that standardizes error handling and logging, reducing boilerplate while ensuring consistent behavior across the codebase.

### Verify

```bash
# Detect unprotected JSON.parse calls on external data
grep -r 'JSON\.parse' src/ | grep -v 'try' | grep -v 'catch' | grep -v '//' | grep -v test

# Detect unprotected parseTOML calls
grep -r 'parseTOML' src/ | grep -v 'try' | grep -v 'catch' | grep -v '//' | grep -v test

# Verify parse error test coverage
npm test -- --grep 'parse.*error|malformed.*json|invalid.*toml'
```

**Accept when:**
- All JSON.parse and parseTOML calls on external data sources in src/ are wrapped in try-catch blocks or documented error boundaries
- Parse error handlers log appropriate context and either propagate errors, use defaults, or terminate gracefully
- Test coverage includes malformed JSON and TOML input scenarios for all configuration loaders and agent implementations
- Exceptions (EXC-001 or EXC-002) are documented in code comments with rationale where applicable

<enforcement>
Clause Code MUST NOT skip or defer verification. All JSON.parse and parseTOML operations on external data must be protected by try-catch blocks or documented error boundaries before merge.
</enforcement>
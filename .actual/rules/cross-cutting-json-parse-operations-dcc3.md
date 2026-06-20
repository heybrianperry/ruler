# Enforce JSON.parse and TOML Parsing with Try-Catch for Configuration and External Data: Json Parse Operations

These rules are ALWAYS ACTIVE for all agent implementations, configuration loaders, MCP path handlers, VSCode settings managers, and any module that parses external data from JSON or TOML sources originating from the file system, environment variables, or external sources.

### Rules

- **R-JSON-001** MUST: All JSON.parse operations on external data sources (file system, environment variables, network responses) MUST be wrapped in try-catch blocks or equivalent error handling mechanisms.
- **R-JSON-002** MUST: All parseTOML operations on external data sources MUST be wrapped in try-catch blocks or equivalent error handling mechanisms.
- **R-JSON-003** SHOULD: For agent initialization code (FirebenderAgent, MistralVibeAgent, GeminiCliAgent), use console.warn for parse failures and continue with default configuration to allow the agent to start even if custom config is malformed.
- **R-JSON-004** SHOULD: For critical configuration loaders (ConfigLoader, UnifiedConfigLoader), propagate parse errors to the caller with context about which file failed, allowing the application to decide whether to fail-fast or fall back to defaults.
- **R-JSON-005** SHOULD: When logging parse errors, include the file path and operation type but avoid logging the raw malformed content which might contain sensitive data or be excessively large.
- **R-JSON-006** MAY: Consider implementing a parseJSON utility function that standardizes error handling and logging, reducing boilerplate while ensuring consistent behavior across the codebase.

### Verify

```bash
# Detect unprotected JSON.parse calls on external data
grep -r 'JSON\.parse' src/ | grep -v 'try' | grep -v 'catch' | grep -v '//' | grep -v test

# Detect unprotected parseTOML calls
grep -r 'parseTOML' src/ | grep -v 'try' | grep -v 'catch' | grep -v '//' | grep -v test

# Run parse error handling tests
npm test -- --grep 'parse.*error|malformed.*json|invalid.*toml'
```

**Accept when:**
- All JSON.parse and parseTOML calls on external data sources in src/ are wrapped in try-catch blocks or documented error boundaries
- Parse error handlers log appropriate context and either propagate errors, use defaults, or terminate gracefully
- Test coverage includes malformed JSON and TOML input scenarios for all configuration loaders and agent implementations
- Verification commands return no unprotected parse operations

<enforcement>
Clause R-JSON-001 and R-JSON-002 are mandatory. Code review MUST verify try-catch wrapping for all external parse operations. CI pipeline MUST run verification commands and flag violations. Claude Code MUST NOT skip or defer verification of these rules.
</enforcement>
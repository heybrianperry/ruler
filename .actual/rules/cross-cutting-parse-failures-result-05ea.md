# Enforce JSON.parse and TOML Parsing with Try-Catch for Configuration and External Data: Parse Failures Result

These rules are ALWAYS ACTIVE for all agent implementations, configuration loaders, MCP path handlers, VSCode settings managers, and any module that parses external data from JSON or TOML sources originating from file system, environment variables, or external sources.

### Rules

- **R-PARSE-001** MUST: Parse failures MUST result in either graceful degradation with default values, explicit error propagation to the caller, or controlled application termination with clear error messages.
- **R-PARSE-002** MUST: All JSON.parse and parseTOML calls on external data sources be wrapped in try-catch blocks or documented error boundaries.
- **R-PARSE-003** MUST: Parse error handlers log appropriate context including file path and operation type, but MUST NOT log raw malformed content which might contain sensitive data or be excessively large.
- **R-PARSE-004** SHOULD: For agent initialization code (FirebenderAgent, MistralVibeAgent, GeminiCliAgent), use console.warn for parse failures and continue with default configuration to allow the agent to start even if custom config is malformed.
- **R-PARSE-005** SHOULD: For critical configuration loaders (ConfigLoader, UnifiedConfigLoader), propagate parse errors to the caller with context about which file failed, allowing the application to decide whether to fail-fast or fall back to defaults.
- **R-PARSE-006** SHOULD: Limit try-catch scope to only the parse operation and immediate result handling, and verify caught exceptions are of expected types (SyntaxError for JSON.parse).
- **R-PARSE-007** MAY: Implement a parseJSON utility function that standardizes error handling and logging, reducing boilerplate while ensuring consistent behavior across the codebase.

### Verify

```bash
# Detect unprotected JSON.parse calls on external data
grep -r 'JSON\.parse' src/ | grep -v 'try' | grep -v 'catch' | grep -v '//' | grep -v test

# Detect unprotected parseTOML calls on external data
grep -r 'parseTOML' src/ | grep -v 'try' | grep -v 'catch' | grep -v '//' | grep -v test

# Run parse error and malformed input tests
npm test -- --grep 'parse.*error|malformed.*json|invalid.*toml'
```

**Accept when:**
- All JSON.parse and parseTOML calls on external data sources in src/ are wrapped in try-catch blocks or documented error boundaries
- Parse error handlers log appropriate context and either propagate errors, use defaults, or terminate gracefully
- Test coverage includes malformed JSON and TOML input scenarios for all configuration loaders and agent implementations
- No unprotected JSON.parse or parseTOML calls are detected by verification commands
- Parse error tests pass for all configuration loaders and agent implementations

<enforcement>
Clause Code MUST NOT skip or defer verification. All JSON.parse and parseTOML operations on external data must be verified to have appropriate error handling before code review approval.
</enforcement>
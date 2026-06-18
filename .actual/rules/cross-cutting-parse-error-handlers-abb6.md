# Enforce JSON.parse and TOML Parsing with Try-Catch for Configuration and External Data: Parse Error Handlers

These rules are ALWAYS ACTIVE for all agent implementations, configuration loaders, MCP path handlers, VSCode settings managers, and any module that parses external data from JSON or TOML sources originating from the file system, environment variables, or external sources.

### Rules

- **R-PARSE-001** MUST: All JSON.parse and parseTOML calls on external data sources MUST be wrapped in try-catch blocks or documented within a higher-level error boundary that catches and handles all exceptions from the parsing operation.

- **R-PARSE-002** MUST: Parse error handlers MUST log sufficient context (file path, operation type) for debugging while avoiding disclosure of sensitive data from the malformed input.

- **R-PARSE-003** MUST: Parse errors MUST be handled separately from schema validation errors, enabling distinct error messages and debugging strategies for each failure mode.

- **R-PARSE-004** SHOULD: For agent initialization code (FirebenderAgent, MistralVibeAgent, GeminiCliAgent), use console.warn for parse failures and continue with default configuration to allow the agent to start even if custom config is malformed.

- **R-PARSE-005** SHOULD: For critical configuration loaders (ConfigLoader, UnifiedConfigLoader), propagate parse errors to the caller with context about which file failed, allowing the application to decide whether to fail-fast or fall back to defaults.

- **R-PARSE-006** SHOULD: Limit try-catch scope to only the parse operation and immediate result handling; verify caught exceptions are of expected types (SyntaxError for JSON.parse).

- **R-PARSE-007** MAY: Consider implementing a parseJSON utility function that standardizes error handling and logging, reducing boilerplate while ensuring consistent behavior across the codebase.

### Verify

```bash
# Detect unprotected JSON.parse calls on external data
grep -r 'JSON\.parse' src/ | grep -v 'try' | grep -v 'catch' | grep -v '//' | grep -v test

# Detect unprotected parseTOML calls on external data
grep -r 'parseTOML' src/ | grep -v 'try' | grep -v 'catch' | grep -v '//' | grep -v test

# Run parse error handling tests
npm test -- --grep 'parse.*error|malformed.*json|invalid.*toml'
```

**Accept when:**
- All JSON.parse and parseTOML calls on external data sources in src/ are wrapped in try-catch blocks or documented error boundaries
- Parse error handlers log appropriate context (file path, operation type) and either propagate errors, use defaults, or terminate gracefully
- Parse error handlers avoid logging raw malformed content that might contain sensitive data or be excessively large
- Test coverage includes malformed JSON and TOML input scenarios for all configuration loaders and agent implementations
- Any exceptions to R-PARSE-001 are documented in code comments with rationale (EXC-001: higher-level error boundary; EXC-002: fail-fast initialization)

<enforcement>
Clause Code MUST NOT skip or defer verification. All JSON.parse and parseTOML operations on external data must be verified to have appropriate error handling before code review approval.
</enforcement>
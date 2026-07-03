# Enforce JSON.parse and TOML Parsing with Try-Catch for Configuration and External Data: Configuration Loaders Separate

These rules are ALWAYS ACTIVE for all agent implementations, configuration loaders, MCP path handlers, VSCode settings managers, and any module that parses external data from JSON or TOML sources originating from file system, environment variables, or external sources.

### Rules

- **R-CONFIG-001** MUST: Wrap all JSON.parse and parseTOML calls on external data sources in try-catch blocks or document them within a higher-level error boundary that catches and handles all exceptions from the parsing operation.
- **R-CONFIG-002** SHOULD: Configuration loaders SHOULD separate parse error handling from schema validation error handling to provide distinct error messages for syntax errors versus semantic errors.
- **R-CONFIG-003** SHOULD: When logging parse errors, include the file path and operation type but avoid logging the raw malformed content which might contain sensitive data or be excessively large.
- **R-CONFIG-004** SHOULD: For agent initialization code (FirebenderAgent, MistralVibeAgent, GeminiCliAgent), use console.warn for parse failures and continue with default configuration to allow the agent to start even if custom config is malformed.
- **R-CONFIG-005** SHOULD: For critical configuration loaders (ConfigLoader, UnifiedConfigLoader), propagate parse errors to the caller with context about which file failed, allowing the application to decide whether to fail-fast or fall back to defaults.
- **R-CONFIG-006** MAY: Consider implementing a parseJSON utility function that standardizes error handling and logging, reducing boilerplate while ensuring consistent behavior across the codebase.

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
- Parse error handlers log appropriate context and either propagate errors, use defaults, or terminate gracefully
- Test coverage includes malformed JSON and TOML input scenarios for all configuration loaders and agent implementations
- Code review checklist verifies try-catch blocks or documented error boundaries for all parsing operations
- Exception cases (EXC-001 or EXC-002) are documented in code comments with rationale

<enforcement>
Clause Code MUST NOT skip or defer verification. All JSON.parse and parseTOML operations on external data must be wrapped in try-catch blocks or documented within error boundaries. Parse error handling must be separated from validation error handling. Verification commands must pass before acceptance.
</enforcement>
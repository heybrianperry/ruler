# Enforce JSON.parse and TOML Parsing with Try-Catch for Configuration and External Data: Modules Implement Retry

These rules are ALWAYS ACTIVE for all agent implementations, configuration loaders, MCP path handlers, VSCode settings managers, and any module that parses external data from JSON or TOML sources originating from file system, environment variables, or external sources.

### Rules

- **R-JSON-TOML-001** MUST: Wrap all JSON.parse and parseTOML calls on external data sources in try-catch blocks or document them within a higher-level error boundary that catches and handles all exceptions from the parsing operation.
- **R-JSON-TOML-002** MUST: Separate parse error handling from schema validation error handling. Parse errors must be caught at the parsing operation level; validation errors may be caught at the Zod schema validation level.
- **R-JSON-TOML-003** MUST: When logging parse errors, include the file path and operation type but avoid logging raw malformed content that might contain sensitive data or be excessively large.
- **R-JSON-TOML-004** SHOULD: For agent initialization code (FirebenderAgent, MistralVibeAgent, GeminiCliAgent), use console.warn for parse failures and continue with default configuration to allow the agent to start even if custom config is malformed.
- **R-JSON-TOML-005** SHOULD: For critical configuration loaders (ConfigLoader, UnifiedConfigLoader), propagate parse errors to the caller with context about which file failed, allowing the application to decide whether to fail-fast or fall back to defaults.
- **R-JSON-TOML-006** MAY: Modules MAY implement retry logic or fallback parsing strategies (e.g., attempting to strip comments before JSON.parse) when appropriate for the use case.
- **R-JSON-TOML-007** MUST NOT: Apply try-catch blocks so broadly that they catch errors beyond parse failures, which could hide bugs in surrounding code. Limit try-catch scope to only the parse operation and immediate result handling.

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
- Exceptions to this rule (EXC-001: parsing within higher-level error boundary; EXC-002: fail-fast initialization) are documented in code comments with rationale

<enforcement>
Clause Code MUST NOT skip or defer verification. All JSON.parse and parseTOML operations on external data must be verified as wrapped or documented before merge. Violations require PR review feedback and remediation.
</enforcement>
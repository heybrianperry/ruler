# Enforce JSON.parse and TOML Parsing with Try-Catch for Configuration and External Data: Toml Parsing Operations

These rules are ALWAYS ACTIVE for all agent implementations, configuration loaders, MCP path handlers, VSCode settings managers, and any module that parses external data from JSON or TOML sources originating from file system, environment variables, or external sources.

### Rules

- **R-TOML-001** MUST: All TOML parsing operations using @iarna/toml or equivalent libraries MUST be wrapped in try-catch blocks or equivalent error handling mechanisms.
- **R-TOML-002** MUST: All JSON.parse operations on data originating from file system, environment variables, or external sources MUST be wrapped in try-catch blocks or equivalent error handling mechanisms.
- **R-TOML-003** MUST: Parse error handlers MUST log appropriate context including the file path and operation type, but MUST NOT log raw malformed content which might contain sensitive data or be excessively large.
- **R-TOML-004** MUST: Parse errors MUST either be propagated to the caller with context about which file failed, use documented default values, or terminate gracefully—the strategy MUST be explicit and appropriate for the module's role.
- **R-TOML-005** SHOULD: Consider implementing a centralized parseJSON or parseTOML utility function that standardizes error handling and logging to reduce boilerplate while ensuring consistent behavior across the codebase.

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
- All JSON.parse and parseTOML calls on external data sources in src/ are wrapped in try-catch blocks or documented error boundaries (EXC-001 or EXC-002)
- Parse error handlers log appropriate context (file path, operation type) without exposing sensitive data or raw malformed content
- Parse errors are either propagated to the caller with context, use documented defaults, or terminate gracefully—the strategy is explicit in code comments
- Test coverage includes malformed JSON and TOML input scenarios for all configuration loaders and agent implementations
- Code review checklist verification confirms error handling is appropriate for each module's role (critical bootstrap vs. optional feature)

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse and parseTOML operations on external data MUST be wrapped in try-catch blocks or documented exceptions. Violations discovered in code review or CI pipeline MUST be remediated before merge.
</enforcement>
# Enforce JSON.parse and TOML Parsing with Error Handling for External Configuration Input: Components Not Pass

These rules are ALWAYS ACTIVE for all agent implementations and configuration loaders that process external input from files or API contracts in JSON and TOML formats.

### Rules

- **R-PARSE-001** MUST_NOT: Components MUST NOT pass unparsed or unvalidated string content directly to configuration application functions (applyRulerConfig, loadUnifiedConfig).
- **R-PARSE-002** MUST: All JSON.parse calls that process external input MUST be wrapped in try-catch blocks that catch SyntaxError and provide meaningful error context including file path.
- **R-PARSE-003** MUST: All parseTOML calls that process external input MUST be wrapped in error handling blocks with file context information.
- **R-PARSE-004** MUST: Public API contracts (api.public.contracts) that accept configuration data MUST validate input before passing to application logic.
- **R-PARSE-005** SHOULD: Create shared utility functions for common parsing patterns to reduce code duplication across agent implementations.
- **R-PARSE-006** SHOULD: Sanitize error messages before logging or displaying to users; implement structured error handling that separates internal details from user-facing messages.

### Verify

```bash
# Check for unprotected JSON.parse calls in agent implementations
grep -r 'JSON\.parse' src/agents/ src/core/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Check for unprotected parseTOML calls
grep -r 'parseTOML' src/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Check for unvalidated input passed to public API contracts
grep -r 'api\.public\.contracts' src/agents/ | xargs grep -l 'JSON\.parse\|parseTOML' | xargs grep -L 'try.*catch' | wc -l | grep -q '^0$'
```

**Accept when:**
- All JSON.parse and parseTOML calls in agent implementations and configuration loaders are wrapped in error handling blocks
- No unvalidated external input is passed directly to applyRulerConfig or loadUnifiedConfig functions
- Code review confirms consistent error handling patterns across all components in policy scope
- Error messages are sanitized and do not expose sensitive file paths or configuration details

<enforcement>
Claude Code MUST NOT skip or defer verification. All parsing operations on external input MUST be protected with error handling before merge.
</enforcement>
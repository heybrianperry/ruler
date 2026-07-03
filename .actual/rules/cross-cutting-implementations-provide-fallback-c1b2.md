# Enforce JSON.parse and TOML Parsing with Error Handling for External Configuration Input: Implementations Provide Fallback

These rules are ALWAYS ACTIVE for all agent implementations and configuration loaders that process external input from files or API contracts, including OpenCodeAgent, MistralVibeAgent, CodexCliAgent, AmazonQCliAgent, and UnifiedConfigLoader.

### Rules

- **R-JSON-001** MUST: Wrap all JSON.parse calls in try-catch blocks that catch SyntaxError and provide meaningful error context including file path.
- **R-TOML-001** MUST: Use @iarna/toml parseTOML function consistently across all TOML parsing operations and handle parsing exceptions with file context.
- **R-EXT-001** MUST: Ensure no unvalidated external input is passed directly to applyRulerConfig or loadUnifiedConfig functions.
- **R-VAL-001** MUST: Implement consistent error handling patterns across all components in policy scope (agent implementations and configuration loaders).
- **R-API-001** MUST: Validate all external input at public API contracts (api.public.contracts) before data flows into application logic.
- **R-FB-001** MAY: Implementations MAY provide fallback or default configurations when parsing fails, if appropriate for the use case.

### Verify

```bash
# Check for unprotected JSON.parse calls in agent implementations and core modules
grep -r 'JSON\.parse' src/agents/ src/core/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Check for unprotected parseTOML calls
grep -r 'parseTOML' src/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Check for unprotected parsing in public API contracts
grep -r 'api\.public\.contracts' src/agents/ | xargs grep -l 'JSON\.parse\|parseTOML' | xargs grep -L 'try.*catch' | wc -l | grep -q '^0$'
```

**Accept when:**
- All JSON.parse and parseTOML calls in agent implementations and configuration loaders are wrapped in error handling blocks
- No unvalidated external input is passed directly to applyRulerConfig or loadUnifiedConfig functions
- Code review confirms consistent error handling patterns across all components in policy scope
- Error messages are sanitized to prevent exposure of sensitive file paths or configuration details
- Parsing validation occurs once at load time rather than on every access

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse and TOML parsing operations must be protected with error handling before code is committed. Violations block merge and require security review for any exceptions.
</enforcement>
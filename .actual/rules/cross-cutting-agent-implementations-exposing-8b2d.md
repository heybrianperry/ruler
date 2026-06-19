# Enforce JSON.parse and TOML Parsing with Error Handling for External Configuration Input: Agent Implementations Exposing

These rules are ALWAYS ACTIVE for all agent implementations and configuration loaders that process external input from files or API contracts, including OpenCodeAgent, MistralVibeAgent, CodexCliAgent, AmazonQCliAgent, and UnifiedConfigLoader.

### Rules

- **R-PARSE-001** MUST: Agent implementations exposing public contracts (api.public.contracts) MUST validate all configuration input before passing to applyRulerConfig or equivalent configuration application functions.
- **R-PARSE-002** MUST: All JSON.parse calls in agent implementations and configuration loaders MUST be wrapped in try-catch blocks that catch SyntaxError and provide meaningful error context including file path.
- **R-PARSE-003** MUST: All parseTOML calls MUST be wrapped in error handling blocks with file context information.
- **R-PARSE-004** MUST: No unvalidated external input from filesystem or API contracts MUST be passed directly to applyRulerConfig or loadUnifiedConfig functions.
- **R-PARSE-005** SHOULD: Create shared utility functions for common parsing patterns to reduce code duplication across agent implementations.
- **R-PARSE-006** SHOULD: Sanitize error messages before logging or displaying to users to prevent exposure of sensitive file paths or configuration details.
- **R-PARSE-007** SHOULD: Implement caching for parsed and validated configurations to avoid repeated validation overhead.

### Verify

```bash
# Check for unprotected JSON.parse calls in agent implementations
grep -r 'JSON\.parse' src/agents/ src/core/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Check for unprotected parseTOML calls
grep -r 'parseTOML' src/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Check that public contracts with parsing have error handling
grep -r 'api\.public\.contracts' src/agents/ | xargs grep -l 'JSON\.parse\|parseTOML' | xargs grep -L 'try.*catch' | wc -l | grep -q '^0$'
```

**Accept when:**
- All JSON.parse and parseTOML calls in agent implementations and configuration loaders are wrapped in error handling blocks
- No unvalidated external input is passed directly to applyRulerConfig or loadUnifiedConfig functions
- Code review confirms consistent error handling patterns across all components in policy scope
- Error messages are sanitized and do not expose sensitive file paths or configuration details
- Shared validation utilities exist for common parsing patterns

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to agent implementations or configuration loaders that handle external input.
</enforcement>